# Etheos Server Execution Model

**Project:** Etheos v0.7.1  
**Status:** VERIFIED (Source code analysis)  
**Last Updated:** 2026-09-22  
**Purpose:** Document actual execution model to inform bug analysis and architectural decisions

---

## Executive Summary

**CRITICAL DISCOVERY:** Etheos uses a **SINGLE-THREADED** execution model, NOT multi-threaded concurrent execution. This fundamentally changes how we must analyze "race conditions" and concurrency bugs.

### Key Findings

1. **Main loop is single-threaded** - One event loop processes all work sequentially
2. **No thread pool** - No concurrent request processing
3. **Timer callbacks unlock mutex** - Allows reentrancy but NOT true parallelism
4. **Sequential packet processing** - Packets processed one at a time
5. **Reentrancy is possible** - Callbacks can trigger other events before completion

### Terminology Impact

- ❌ **INCORRECT:** "Race condition" (implies concurrent execution)
- ✅ **CORRECT:** "Reentrancy issue", "delayed state update", "non-atomic operation"

---

## Main Loop Architecture

### Single-Threaded Event Loop

**File:** `src/main.cpp:461-468`

```cpp
// SINGLE THREADED MAIN LOOP
while (eoserv_running)
{
    server->Tick();  // Process all events sequentially
    
    // Signal handling
    if (eoserv_sig_rehash) {
        // Reload config
    }
}
```

### Server::Tick() Execution

The `Server::Tick()` method processes work in **strict sequential order**:

1. **Timer events** - `timer.Tick()` fires expired timers
2. **Network I/O** - Accept new connections, read packets
3. **Packet processing** - Handle each packet sequentially
4. **World updates** - NPC AI, item respawns, etc.
5. **Database operations** - Execute queued queries

**CRITICAL:** All work happens in ONE thread. No concurrent execution.

---

## Timer System Architecture

### Timer::Tick() Execution Model

**File:** `src/timer.cpp:182-257`

```cpp
void Timer::Tick()
{
    double currenttime = Timer::GetTime();
    
    impl->lock();  // ← LOCK ACQUIRED
    
    if (this->changed) {
        this->execlist = this->timers;
        this->changed = false;
    }
    
    UTIL_FOREACH(this->execlist, timer) {
        if (this->timers.find(timer) == this->timers.end())
            continue;
        
        if (timer->lasttime + timer->speed < currenttime) {
            impl->unlock();  // ← UNLOCK BEFORE CALLBACK
            
            timer->lasttime += timer->speed;
            
            if (timer->lifetime != Timer::FOREVER) {
                --timer->lifetime;
                
                if (timer->lifetime == 0) {
                    this->Unregister(timer);
                }
            }
            
            try {
                timer->callback(timer->param);  // ← CALLBACK EXECUTES UNLOCKED
            }
            catch (...) {
                Console::Err("Timer callback caused an exception");
            }
            
            if (timer->manager == 0)
                delete timer;
            
            impl->lock();  // ← LOCK REACQUIRED
        }
    }
    
    impl->unlock();  // ← FINAL UNLOCK
}
```

### Critical Observations

#### 1. Mutex Purpose
The mutex protects **timer registration**, NOT callback execution:
- Lock held when iterating `execlist`
- Lock **released** during callback
- Lock **reacquired** after callback completes

#### 2. Reentrancy Window
While callback executes:
- ❌ Lock is NOT held
- ✅ Callback can call `Timer::Register()` (acquires lock)
- ✅ Callback can call `Timer::Unregister()` (acquires lock)
- ✅ Callback can trigger character warps
- ✅ Warp can trigger other callbacks

#### 3. NOT True Concurrency
- ⚠️ Only ONE callback executes at a time
- ⚠️ No parallel threads running callbacks
- ⚠️ Reentrancy is **nested calls**, not **concurrent calls**

---

## Packet Processing Model

### Sequential Processing

**File:** `src/eoserver.cpp` (Server::Tick)

Packets are processed **one at a time** in the order received:

```cpp
// Simplified pseudocode
for (each client) {
    for (each packet in recv_buffer) {
        ProcessPacket(client, packet);  // Sequential
    }
}
```

### No Concurrent Attacks

When two players attack "simultaneously":

```
Client A sends PACKET_ATTACK at time T
Client B sends PACKET_ATTACK at time T

Server receives:
  Packet A arrives first  → Processed completely
  Packet B arrives second → Processed completely

BOTH PACKETS PROCESSED BEFORE ANY WARP EXECUTES
```

This is **NOT** a race condition (no concurrent execution).  
This is a **reentrancy issue** (delayed state updates).

---

## Arena Attack Flow

### Single-Threaded Attack Processing

**File:** `src/arena.cpp:120-210`

```cpp
void Arena::Attack(Character *from, Direction direction)
{
    // 1. Calculate target coordinates
    int target_x = from->x;
    int target_y = from->y;
    // ... adjust based on direction
    
    // 2. Find victim at target tile
    UTIL_FOREACH(this->map->characters, character) {
        if (character->arena == this
            && character->x == target_x
            && character->y == target_y) {
            
            // 3. Increment kill count
            ++from->arena_kills;
            
            // 4. QUEUE warp (doesn't execute yet)
            actions.push_back({character, map, x, y});
            
            // 5. Check match end condition
            if (from->arena->occupants == 2) {
                // Both attacks see occupants == 2
                actions.push_back({from, map, x, y});
            }
            
            break;
        }
    }
    
    // 6. EXECUTE queued warps (too late to prevent issue)
    UTIL_FOREACH(actions, act) {
        act.character->Warp(...);
    }
}
```

### The "Simultaneous Attack" Scenario

```
Initial state: PlayerA and PlayerB in arena (occupants = 2)

Server::Tick() begins:

1. Process PACKET_ATTACK from PlayerA
   - Enter Arena::Attack(PlayerA)
   - Find PlayerB at target
   - ++PlayerA->arena_kills
   - Queue warp for PlayerB
   - Check: occupants == 2? YES
   - Queue warp for PlayerA (winner!)
   - Execute warps:
     - PlayerB->Warp() → --arena->occupants (now 1)
     - PlayerB->arena = NULL
     - PlayerA->Warp() → --arena->occupants (now 0)
     - PlayerA->arena = NULL
   - Exit Arena::Attack(PlayerA)

2. Process PACKET_ATTACK from PlayerB
   - Enter Arena::Attack(PlayerB)
   - Find PlayerA at target... WAIT
   - PlayerA->arena == NULL (was set to NULL above)
   - if (character->arena == this) → FALSE
   - PlayerB finds NO victim
   - Exit Arena::Attack(PlayerB)

RESULT: Only PlayerA declared winner (correct!)
```

**WAIT... This suggests Bug #1 might NOT actually happen!**

Let me re-analyze:

### Re-Analysis: Can Bug #1 Actually Occur?

The bug claims both attacks see `occupants == 2`. Let's trace:

```
Initial: occupants = 2, PlayerA->arena = arena, PlayerB->arena = arena

Attack 1 (PlayerA attacks PlayerB):
  Line 150: if (character->arena == this) → TRUE (PlayerB->arena == arena)
  Line 160: ++from->arena_kills
  Line 161: actions.push_back({PlayerB, ...})
  Line 178: if (from->arena->occupants == 2) → TRUE
  Line 180:   actions.push_back({PlayerA, ...})
  Line 205-207: EXECUTE WARPS
    - PlayerB->Warp() → PlayerB->arena = NULL, --occupants (now 1)
    - PlayerA->Warp() → PlayerA->arena = NULL, --occupants (now 0)

Attack 2 (PlayerB attacks PlayerA):
  Line 150: if (character->arena == this) → FALSE (PlayerA->arena == NULL)
  - NO MATCH FOUND
  - NO ACTIONS
```

**CONCLUSION:** Bug #1 does NOT occur with sequential processing!

The warps execute INSIDE Arena::Attack() before it returns, so the second attack sees the updated state.

### But... What If Warp Is Async?

Let me check `Character::Warp()`:

---

## Character Warp Execution

**File:** `src/character.cpp:1296-1308`

```cpp
void Character::Warp(short map, unsigned char x, unsigned char y, WarpAnimation anim)
{
    // SYNCHRONOUS, NOT QUEUED
    
    if (this->arena) {
        --this->arena->occupants;  // IMMEDIATE
        this->arena = 0;            // IMMEDIATE
    }
    
    // ... actual warp logic
    
    if (this->next_arena) {
        this->arena = this->next_arena;
        ++this->arena->occupants;   // IMMEDIATE
        this->next_arena = 0;
    }
}
```

**CONFIRMED:** Warps are **synchronous** and state updates are **immediate**.

---

## Revised Bug #1 Analysis

### The Real Issue

Bug #1 as described **cannot happen** with synchronous warps. However, a different issue CAN occur:

### Actual Bug: Action Queue Decoupling

The `actions` vector decouples victim identification from warp execution:

```cpp
// Phase 1: Identify victims (state is X)
UTIL_FOREACH(this->map->characters, character) {
    if (character->arena == this && ...) {
        actions.push_back({character, ...});
    }
}

// Phase 2: Execute warps (state is X + delays)
UTIL_FOREACH(actions, act) {
    act.character->Warp(...);
}
```

**Problem:** If callback occurs BETWEEN Phase 1 and Phase 2:

```
1. Arena::Attack(PlayerA) begins
2. Phase 1: Find PlayerB, add to actions
3. Timer callback fires (reentrancy)
4. Timer → Arena::Spawn() → Warp players → State changes
5. Return from callback
6. Phase 2: Execute queued warps with STALE state
```

**THIS** is the actual reentrancy bug!

---

## Reentrancy Scenarios

### Scenario 1: Timer During Attack

```
Arena::Attack(PlayerA)
  → Find victim
  → Queue action
  → REENTRANT CALL (timer fires)
    → Arena::Spawn()
      → Warp new players in
      → Modify map->characters
      → Return
  → Execute queued action (stale character pointer)
```

### Scenario 2: Warp During Warp

```
Arena::Attack(PlayerA)
  → actions.push_back({PlayerB, ...})
  → actions.push_back({PlayerC, ...})
  → EXECUTE actions:
    → PlayerB->Warp()
      → Character::Warp() enters
      → Send packets
      → REENTRANT CALL (packet handler)
        → Player attacks
        → Arena::Attack() nested call
        → Return
      → Continue warp
      → Return
    → PlayerC->Warp()
```

---

## Concurrency vs Reentrancy

### True Concurrency (NOT in Etheos)

```
Thread 1: Arena::Attack(PlayerA) ─────────────►
Thread 2: Arena::Attack(PlayerB) ─────────────►
          ↑
          Race condition: both see same state
```

### Reentrancy (Actual Etheos Model)

```
Main Thread:
  Arena::Attack(PlayerA)
    ├─ Find victim
    ├─ Queue action
    ├─ Timer::Tick() fires
    │   ├─ unlock() mutex
    │   ├─ Arena::Spawn() ◄─── NESTED CALL
    │   │   └─ Modify state
    │   ├─ Return
    │   └─ lock() mutex
    └─ Execute action (stale state)
```

**Key Difference:**
- Concurrency: Two threads running **at the same time**
- Reentrancy: One thread, **nested function calls**

---

## Implications for Bug Analysis

### Bugs Affected by Execution Model

| Bug | Original Classification | Revised Classification |
|-----|-------------------------|------------------------|
| #1 | Race condition | Reentrancy / delayed state update |
| #2 | State desync | Multiple decrement paths (still valid) |
| #3 | Validation | Input validation (still valid) |
| #4 | Use-after-free | Timer lifetime (more complex) |
| #5 | Overflow | Data type (still valid) |
| #6 | Logic flaw | Design issue (still valid) |
| #7 | Logic flaw | Design issue (still valid) |
| #8 | UX issue | Broadcast spam (still valid) |

### Bug #4 Revisited: Timer Destruction

With single-threaded model:

```
Server::Tick()
  → Timer::Tick()
    → Unlock mutex
    → arena_spawn(arena_ptr) ◄─── IN CALLBACK
      → Arena::Spawn()
        → Character::Warp()
          → Map method call
            → MAP RELOAD TRIGGERED (admin command?)
              → delete old_arena ◄─── DESTROYS Arena
                → ~Arena()
                  → delete spawn_timer
              → new Arena created
          → Return from warp
      → Return from Arena::Spawn()
    → impl->lock()  ◄─── REACQUIRES LOCK
    → NEXT LINE: if (timer->manager == 0) delete timer
      → Double free? Use after free?
```

**The actual risk:**
- Timer deleted in ~Arena()
- Callback STILL EXECUTING
- Callback accesses `arena->Spawn()` on freed memory

**BUT:** With single-threaded model, this requires **reentrancy**, not concurrency.

Map reload must happen DURING the callback, not in parallel.

---

## Testing Implications

### Don't Test for Concurrency

❌ **WRONG:**
```
Test: Spawn 2 threads, both call Arena::Attack() simultaneously
```

✅ **RIGHT:**
```
Test: Trigger timer callback during Arena::Attack()
Verify: Stale state handling
```

### Don't Use Thread Sanitizers (Directly)

❌ **ThreadSanitizer** - Won't find issues (no threads)  
✅ **AddressSanitizer** - Will find use-after-free  
✅ **Valgrind** - Will find memory errors  
✅ **Reentrancy Testing** - Manual trigger of nested calls  

---

## Architectural Recommendations

### For Single-Threaded Model

1. **State Updates:** Make atomic within function scope
2. **Callbacks:** Avoid modifying state during iteration
3. **Reentrancy Guards:** Detect and prevent nested calls
4. **Validity Checks:** Verify pointers before use
5. **Deferred Actions:** Queue actions, validate before execute

### Example: Reentrancy Guard

```cpp
class Arena {
    bool in_attack = false;
    
    void Attack(Character *from, Direction direction) {
        if (in_attack) {
            Console::Err("Reentrancy detected in Arena::Attack");
            return;  // Prevent nested call
        }
        
        in_attack = true;
        
        // ... attack logic
        
        in_attack = false;
    }
};
```

---

## Summary

**Etheos v0.7.1 Execution Model:**
- ✅ Single-threaded main loop
- ✅ Sequential packet processing
- ✅ Timer callbacks can reenter
- ✅ No concurrent execution
- ❌ NOT multi-threaded
- ❌ NOT parallel processing

**Terminology Corrections:**
- ❌ "Race condition" → ✅ "Reentrancy issue"
- ❌ "Thread-safe" → ✅ "Reentrant-safe"
- ❌ "Concurrent access" → ✅ "Nested access"

**Bug Analysis Impact:**
- Bug #1 requires complete re-analysis
- Bug #4 is reentrancy-based, not concurrency-based
- Testing strategy must focus on reentrancy, not concurrency

---

## References

- `src/main.cpp:461-468` - Main loop
- `src/timer.cpp:182-257` - Timer::Tick()
- `src/arena.cpp` - Arena implementation
- `src/character.cpp:1296-1308` - Character::Warp()

---

**Next Steps (Phase 0.5):**
1. ✅ EXECUTION-MODEL.md created
2. ⏭️ Update LEGACY-ARENA-BUGS.md with corrected terminology
3. ⏭️ Re-verify all 8 bugs with single-threaded model
4. ⏭️ Create research baseline documentation
