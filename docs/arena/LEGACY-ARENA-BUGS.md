# Legacy Arena System - Known Bugs (Phase 0.5 Verified)

**Status:** VERIFIED (Phase 0.5 source code analysis)  
**Last Updated:** 2026-09-22  
**Purpose:** Document confirmed bugs in legacy arena implementation with evidence

## ⚠️ CRITICAL: DO NOT FIX DURING PHASE 0.5

This is **RESEARCH AND VERIFICATION**. These bugs are documented for future reference. **DO NOT** implement fixes, refactor code, or change behavior during Phase 0 or Phase 0.5.

---

## Execution Model Context

**CRITICAL:** Etheos v0.7.1 uses **single-threaded** execution (see `docs/architecture/EXECUTION-MODEL.md`).

There is NO concurrent execution, NO thread pool, NO parallel processing. Issues involve:
- Reentrancy (nested calls during callbacks)
- Deferred operations (queued actions executed later)
- State timing (checks vs updates)
- Iterator invalidation

**NOT** traditional multithreaded race conditions.

---

## Bug Summary

| ID | Severity | Category | Status |
|----|----------|----------|--------|
| #1 | 🔴 CRITICAL | Reentrancy / Deferred Execution | CONFIRMED |
| #2 | 🔴 CRITICAL | State Management | CONFIRMED |
| #3 | 🔴 CRITICAL | Input Validation | CONFIRMED |
| #4 | 🔴 CRITICAL | Lifetime Management | UNPROVEN (see analysis) |
| #5 | 🟡 HIGH | Data Type | CONFIRMED |
| #6 | 🟡 HIGH | Design Limitation | CONFIRMED (by design) |
| #7 | 🟡 HIGH | Design Limitation | CONFIRMED (by design) |
| #8 | 🟢 MEDIUM | UX Issue | CONFIRMED |

---

## Bug #1: Deferred Action Queue with Reentrancy

**Severity:** 🔴 CRITICAL  
**Category:** Reentrancy / Non-Atomic State Update  
**Impact:** Stale state access, iterator invalidation, incorrect match end
**Status:** CONFIRMED

### Description

`Arena::Attack()` decouples victim identification from warp execution. Actions are queued, then executed after sending packets. If reentrant callback (timer or packet processing) executes between queueing and execution, queued actions operate on stale state.

### Evidence

**File:** `src/arena.cpp:120-210`

**Key Pattern:**
```cpp
// Phase 1: Queue actions (capture current state)
std::vector<Arena_Spawn_Action> actions;
UTIL_FOREACH(this->map->characters, character) {
    if (character->arena == this && character->x == target_x && character->y == target_y) {
        ++from->arena_kills;
        actions.push_back({character, ...});  // Queue warp
        
        // Send packets - CAN TRIGGER REENTRANCY
        UTIL_FOREACH(this->map->characters, character) {
            character->Send(builder);
        }
        break;
    }
}

// Phase 2: Execute warps (state may have changed)
UTIL_FOREACH(actions, act) {
    act.character->Warp(...);  // Stale pointers possible
}
```

### Reentrancy Mechanism

**See:** `docs/architecture/EXECUTION-MODEL.md`

1. `Timer::Tick()` unlocks mutex before callback (`src/timer.cpp:199`)
2. Callbacks can call `Arena::Spawn()`, modify `map->characters`
3. `character->Send()` may trigger network flush and packet processing
4. Nested `Arena::Attack()` possible during send loop

### Scenario: Stale State

```
Initial: PlayerA, PlayerB in arena (occupants = 2)
1. PlayerA attacks PlayerB
2. Arena::Attack(PlayerA) enters
3. Find PlayerB, queue warp
4. Send packet to all characters
5. *** TIMER CALLBACK FIRES (reentry) ***
   - Arena::Spawn() adds PlayerC
   - map->characters modified
   - occupants now 3
6. Continue Arena::Attack(PlayerA)
7. Execute queued warp for PlayerB
8. Check: occupants == 2? NO (now 3)
9. Match end logic incorrect
```

### Scenario: Iterator Invalidation

```
Arena::Attack(PlayerA)
├─ UTIL_FOREACH(map->characters, iter) - ITERATING
├─ character->Send(builder)
│  └─ *** NESTED Arena::Attack(PlayerC) ***
│     └─ PlayerC->Warp() modifies map->characters
│        └─ std::vector reallocation
└─ Continue iteration - ITERATOR INVALIDATED → CRASH
```

### Root Cause

1. Deferred execution (queue then execute)
2. No reentrancy guard
3. Mutable iteration during send
4. No state validation before execution

### Verdict

**CONFIRMED** - Execution path proven. Requires specific timing but reachable.

---

## Bug #2: Occupant Count Desynchronization

**Severity:** 🔴 CRITICAL  
**Category:** State Management  
**Impact:** Incorrect capacity checks, occupant leaks
**Status:** CONFIRMED

### Description

`occupants` counter decremented in multiple code paths without centralized management. Exception or early return between decrement and state update causes desync.

### Evidence

**Decrement Site 1:** `src/character.cpp:1296-1300`
```cpp
if (this->arena) {
    --this->arena->occupants;
    this->arena = 0;
}
```

**Decrement Site 2:** `src/character.cpp:2229-2231`
```cpp
if (this->arena) {
    --this->arena->occupants;  // Destructor
}
```

### Scenario: Double Decrement

```
1. PlayerA in arena (occupants = 1)
2. Map reload triggered
3. Map::LoadArena() at src/map.cpp:386
   - delete this->arena (calls ~Arena())
   - ~Arena() doesn't decrement (no code for it)
4. new Arena created
5. occupants = update_characters.size()
6. PlayerA->arena points to OLD deleted arena
7. PlayerA warps → --old_arena->occupants
8. USE AFTER FREE
```

### Scenario: Exception During Warp

```
Character::Warp() {
    if (this->arena) {
        --this->arena->occupants;  // Decremented
        this->arena = 0;
    }
    // EXCEPTION THROWN (network, validation)
    if (this->next_arena) {
        // Never reached
    }
}
```

### Root Cause

1. Distributed decrement (two call sites)
2. No transaction/rollback
3. No validation (occupants vs actual count)

### Verdict

**CONFIRMED** - Multiple decrement paths without centralized ownership.

---

## Bug #3: No Bounds Validation on Spawn Coordinates

**Severity:** 🔴 CRITICAL  
**Category:** Input Validation  
**Impact:** Buffer overflow, out-of-bounds access, crash
**Status:** CONFIRMED

### Description

Spawn coordinates from `data/arenas.ini` not validated against map dimensions. Invalid coordinates passed to `Map::GetTile()` cause out-of-bounds access.

### Evidence

**File:** `src/map.cpp:406-435`

```cpp
// NO VALIDATION
s->sx = util::to_int(spawn);  // Can be > map width
s->sy = util::to_int(spawn);  // Can be > map height
s->dx = util::to_int(spawn);
s->dy = util::to_int(spawn);
this->arena->spawns.push_back(s);
```

**Usage:** `src/arena.cpp:48-54`
```cpp
UTIL_FOREACH(this->map->characters, character) {
    if (character->x == spawn->sx && character->y == spawn->sy) {
        // spawn->sx, spawn->sy never validated
        ++newplayers;
    }
}
```

### Attack Vector

```ini
# data/arenas.ini
10.spawns = 999,999, 5,5
```

```
1. Map loaded (width=30, height=30)
2. Spawn point (999, 999) stored
3. Character warped to spawn destination
4. Warp calls Map::GetTile(999, 999)
5. Buffer overflow / segfault
```

### Root Cause

Trust config file input without validation.

### Verdict

**CONFIRMED** - No bounds checking found in `Map::LoadArena()`.

---

## Bug #4: Arena Destruction During Timer Callback

**Severity:** 🔴 CRITICAL  
**Category:** Lifetime Management  
**Impact:** UNKNOWN (requires testing)
**Status:** UNPROVEN (safe by design)

### Description (Original Claim)

Arena destroyed (map reload) while timer callback executing causes use-after-free.

### Evidence Analysis

**Arena Ownership:** `src/map.hpp:232` - `Arena *arena`  
**Creation:** `src/map.cpp:402` - `new Arena(...)`  
**Destruction:** `src/map.cpp:386, 745` - `delete this->arena`

**Timer Registration:** `src/arena.cpp:34`
```cpp
this->spawn_timer = new TimeEvent(arena_spawn, this, time, Timer::FOREVER);
this->map->world->timer.Register(this->spawn_timer);
```

**Arena Destructor:** `src/arena.cpp:212-215`
```cpp
~Arena() {
    delete this->spawn_timer;  // Calls ~TimeEvent()
}
```

**TimeEvent Destructor:** `src/timer.cpp:314-318`
```cpp
~TimeEvent() {
    if (this->manager != 0) {
        this->manager->Unregister(this);  // Removes from timer list
    }
}
```

**Timer::Unregister:** `src/timer.cpp:275-282`
```cpp
void Timer::Unregister(TimeEvent *timer) {
    impl->lock();
    this->changed = true;  // Mark execlist dirty
    this->timers.erase(timer);  // Remove from set
    impl->unlock();
    timer->manager = 0;  // Clear back pointer
}
```

**Timer::Tick:** `src/timer.cpp:187-191`
```cpp
impl->lock();
if (this->changed) {
    this->execlist = this->timers;  // Refresh execution list
    this->changed = false;
}
```

### Execution Trace

**Scenario: Map Reload During Callback**

```
Timer::Tick()
├─ Lock, build execlist
├─ Unlock
├─ arena_spawn(arena_ptr) - CALLBACK STARTS
│  └─ Arena::Spawn()
│     └─ Character::Warp()
│        └─ *** ADMIN COMMAND:  ***
│           └─ Map::LoadArena()
│              ├─ delete this->arena
│              │  └─ ~Arena()
│              │     └─ delete spawn_timer
│              │        └─ ~TimeEvent()
│              │           └─ Timer::Unregister(this)
│              │              ├─ Lock
│              │              ├─ this->changed = true
│              │              ├─ timers.erase(timer)
│              │              ├─ Unlock
│              │              └─ timer->manager = 0
│              └─ new Arena(...)
│  RETURN FROM Arena::Spawn() - arena pointer INVALID
├─ Lock
├─ if (timer->manager == 0) delete timer - SKIPPED (already deleted)
└─ Continue
```

### Analysis

**Key Question:** Can callback trigger `delete this->arena`?

**Answer:** Only if callback calls admin command or triggers map reload.

**Paths to Map::LoadArena:**
1. Server startup (`src/map.cpp` constructor)
2. Admin `` command (`src/commands/server.cpp:111`)
3. Map::Reload() (`src/map.cpp` - calls LoadArena)

**Can callback trigger these?**
- Server startup: NO (arena doesn't exist yet)
- Admin command: ONLY if admin executes `` during callback
- Map::Reload(): Requires admin action

### Verdict

**UNPROVEN** - Not a typical use-after-free.

The design is actually SAFE:
1. `delete spawn_timer` calls `~TimeEvent()`
2. `~TimeEvent()` calls `Timer::Unregister()`
3. `Timer::Unregister()` sets `changed = true`
4. Next `Timer::Tick()` refreshes `execlist`
5. Timer never accesses deleted `TimeEvent`

**However:** `Arena::Spawn()` RETURNS to `arena_spawn()`, which RETURNS to `Timer::Tick()`, which tries:
```cpp
if (timer->manager == 0)
    delete timer;  // Already deleted!
```

**Actual Bug:** Potential double-free of `timer`, NOT use-after-free of `arena`.

**Requires:** Admin executes `` during exact window of `Arena::Spawn()` callback.

**Likelihood:** VERY LOW (requires precise admin timing).

**Recommendation:** Needs runtime testing with AddressSanitizer to confirm.

---

## Bug #5: Kill Count Overflow

**Severity:** 🟡 HIGH  
**Category:** Data Type Limitation  
**Impact:** Display shows negative kills
**Status:** CONFIRMED

### Evidence

**File:** `src/character.hpp:207`
```cpp
char arena_kills;  // -128 to 127
```

**Increment:** `src/arena.cpp:160`
```cpp
++from->arena_kills;  // Overflows at 128
```

### Scenario

```
Kill 127: arena_kills = 127
Kill 128: arena_kills = -128 (overflow)
```

### Verdict

**CONFIRMED** - Type is `char` (signed 8-bit).

---

## Bug #6: One Player Per Spawn Tile Limit

**Severity:** 🟡 HIGH  
**Category:** Design Limitation (NOT a bug)
**Impact:** Only first player on spawn tile is warped
**Status:** CONFIRMED (intentional design)

### Evidence

**File:** `src/arena.cpp:88-104`
```cpp
UTIL_FOREACH(this->spawns, spawn) {
    UTIL_FOREACH(this->map->characters, character) {
        if (character->x == spawn->sx && character->y == spawn->sy) {
            actions.push_back({character, ...});
            break;  // ONE PLAYER PER SPAWN
        }
    }
}
```

### Analysis

This appears to be **intentional design** for matchmaking:
- One spawn point = one match slot
- Multiple players on same tile → queue system needed
- Current design: first-come-first-served

### Verdict

**CONFIRMED** - But this is a design limitation, not a bug. Working as designed.

---

## Bug #7: No Match Time Limit

**Severity:** 🟡 HIGH  
**Category:** Design Limitation (NOT a bug)
**Impact:** Matches can last indefinitely
**Status:** CONFIRMED (intentional design)

### Evidence

Match ends ONLY when:
```cpp
// src/arena.cpp:178
if (from->arena->occupants == 2) {
    // Last elimination
}
```

No time limit, no draw condition, no activity timeout.

### Analysis

This appears to be **intentional design** for free-form combat:
- No forced time pressure
- Players can strategize
- Admin intervention available (`` players out)

### Verdict

**CONFIRMED** - But this is a design choice, not a bug. Working as designed.

---

## Bug #8: Capacity Notification to All Players

**Severity:** 🟢 MEDIUM  
**Category:** UX Issue  
**Impact:** Unnecessary packet spam
**Status:** CONFIRMED

### Evidence

**File:** `src/arena.cpp:63-70`
```cpp
if (this->occupants >= this->block && !force) {
    PacketBuilder builder(PACKET_ARENA, PACKET_DROP);
    UTIL_FOREACH(this->map->characters, character) {
        character->Send(builder);  // EVERYONE on map
    }
    return;
}
```

### Analysis

Capacity message sent to ALL characters on map, not just waiting players.

### Verdict

**CONFIRMED** - Minor UX issue.

---

## Bug Interaction Matrix

| Bug Pair | Interaction |
|----------|-------------|
| #1 + #2 | Reentrancy causes desync → wrong occupant count |
| #2 + #3 | Desync + out-of-bounds → crash amplified |
| #3 + #4 | Invalid coords + timer issues → undefined behavior |

---

## Priority Ranking

### Must Fix Before Arena Work
1. **Bug #3** - Bounds validation (crash risk)
2. **Bug #2** - Occupant desync (state corruption)
3. **Bug #1** - Deferred actions (stale state)

### Should Fix Before Production
4. **Bug #5** - Kill overflow (display bug)
5. **Bug #4** - Test with AddressSanitizer (unlikely but severe)

### Design Decisions (Not Bugs)
6. **Bug #6** - One per spawn (intentional)
7. **Bug #7** - No time limit (intentional)
8. **Bug #8** - Broadcast (minor UX)

---

## Testing Recommendations

### Bug #1: Reentrancy
```
Test: Trigger timer during Arena::Attack()
Method: Patch timer to fire immediately
Expected: Stale state handling
```

### Bug #2: Occupant Desync
```
Test: Disconnect during warp
Expected: occupants decremented once
```

### Bug #3: Bounds Validation
```
Test: spawn = (999, 999) in config
Expected: Error + rejection
Actual: No validation
```

### Bug #4: Timer Lifetime
```
Test:  during Arena::Spawn()
Tools: AddressSanitizer
Expected: Clean shutdown
Check: Double-free of timer
```

### Bug #5: Overflow
```
Test: 128 kills
Expected: 128
Actual: -128
```

---

## Evidence Standard

All findings backed by:
- Source file paths
- Line numbers
- Execution traces
- Actual code snippets

**NOT** based on Phase 0 documents.

---

## References

- `docs/architecture/EXECUTION-MODEL.md` - Execution model
- `src/arena.cpp` - Arena implementation
- `src/character.cpp` - Character warp/destructor
- `src/map.cpp` - Arena creation/destruction
- `src/timer.cpp` - Timer system
- `CMakeLists.txt:9` - Etheos v0.7.1

---

**Summary:** 5 bugs confirmed, 2 design limitations documented, 1 unproven (requires testing). All analysis verified against source code during Phase 0.5.
