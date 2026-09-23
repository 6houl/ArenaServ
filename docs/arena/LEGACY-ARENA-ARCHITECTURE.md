# Legacy Arena System Architecture

**Status:** CONFIRMED (ArenaServ source code analysis)  
**Last Updated:** 2026-09-22  
**Purpose:** Complete documentation of existing arena implementation

## ⚠️ WARNING: CRITICAL BUGS IDENTIFIED

This document describes the **EXISTING** legacy arena system for archaeological purposes only. Multiple critical bugs have been identified (see LEGACY-ARENA-BUGS.md). **DO NOT** extend this system without addressing these issues first.

---

## Overview

The legacy arena system provides automatic periodic PvP matches on designated maps. Players stand on spawn tiles, wait for a timer, and are teleported into an arena where attacks eliminate opponents.

**Design Philosophy:** Minimal, timer-based spawning system with kill tracking.

---

## Core Components

### 1. Arena Class

**File:** `src/arena.hpp`, `src/arena.cpp`

```cpp
class Arena {
public:
    int occupants;          // Current players in arena
    int time;               // Spawn interval (seconds)
    int block;              // Maximum occupants before blocking new spawns
    
    TimeEvent *spawn_timer; // Timer for periodic spawns
    Map *map;               // Arena map reference
    std::vector<Arena_Spawn *> spawns; // Spawn point definitions
    
    Arena(Map *map, int time, int block);
    void Spawn(bool force = false);
    void Attack(Character *from, Direction direction);
    ~Arena();
};
```

**Evidence:** `src/arena.hpp:25-44`

### 2. Arena_Spawn Structure

**Purpose:** Defines spawn point pairs (source → destination)

```cpp
struct Arena_Spawn {
    unsigned char sx;  // Source X coordinate (waiting area)
    unsigned char sy;  // Source Y coordinate (waiting area)
    unsigned char dx;  // Destination X (arena combat area)
    unsigned char dy;  // Destination Y (arena combat area)
};
```

**Evidence:** `src/arena.hpp:14-19`

### 3. Character Arena State

**File:** `src/character.hpp`

```cpp
class Character {
    // Arena-related members
    Arena *next_arena;     // Queued arena (set before warp)
    Arena *arena;          // Current arena (active participation)
    char arena_kills;      // Kill count in current match
    // ...
};
```

**Evidence:** `src/character.hpp:205-207`

---

## Configuration

### Arena Definition File

**Location:** `data/arenas.ini` (or override via `config/files.ini`)  
**Format:** INI file with per-map configuration

```ini
# Enable arena on map 10
10.enabled = yes
10.time = 30                           # Spawn every 30 seconds
10.block = 3                           # Max 3 players in arena
10.spawns = 11,9, 7,11, 11,6, 4,4     # Spawn definitions

# Format: sx,sy, dx,dy, sx,sy, dx,dy, ...
# Each quartet defines one spawn point pair
```

**Evidence:** `data/arenas.ini:1-27`

### Spawn Definition Format

```
spawns = sx1,sy1, dx1,dy1, sx2,sy2, dx2,dy2, ...
```

**Rules:**
- Must have multiple of 4 values (quartets)
- Each quartet: `source_x, source_y, dest_x, dest_y`
- Players standing at (sx, sy) warp to (dx, dy)
- Multiple spawn points supported

**Evidence:** `src/map.cpp:406-435`

### Loading Process

```cpp
void Map::LoadArena() {
    // Parse config: world->arenas_config[map_id + ".enabled"]
    // Parse time: world->arenas_config[map_id + ".time"]
    // Parse block: world->arenas_config[map_id + ".block"]
    // Parse spawns: world->arenas_config[map_id + ".spawns"]
    
    // Create Arena instance
    // Parse spawn quartets, create Arena_Spawn objects
    // Register timer event
}
```

**Evidence:** `src/map.cpp:372-448`

---

## State Machine

### Character States

```
NULL (no arena)
    ↓
[Stand on spawn tile]
    ↓
NEXT_ARENA (queued, waiting for timer)
    ↓
[Timer fires, Spawn() called]
    ↓
[Warp triggered]
    ↓
ARENA (active participant)
    ↓
[Eliminated OR Match Ends]
    ↓
[Warp to relog point]
    ↓
NULL (no arena)
```

### Arena Lifecycle

```
[Map Loads]
    ↓
Arena::Arena() - Constructor
    ↓
Register timer (spawn every N seconds)
    ↓
[LOOP FOREVER]
    ↓
Timer fires → Arena::Spawn()
    ↓
Check spawn tiles for waiting players
    ↓
If players found:
    ├→ Check occupants < block (capacity)
    ├→ Set character->next_arena
    ├→ Reset character->arena_kills
    ├→ Warp characters to arena
    └→ Increment occupants
    ↓
[Players fight via Arena::Attack()]
    ↓
[On elimination, decrement occupants]
    ↓
[LOOP CONTINUES]
```

---

## Core Operations

### 1. Spawn Process

**Function:** `Arena::Spawn(bool force)`  
**Trigger:** Timer event (every `time` seconds) OR admin command  
**Purpose:** Teleport waiting players into arena

#### Algorithm

```cpp
void Arena::Spawn(bool force) {
    // 1. Count players on spawn tiles
    int newplayers = 0;
    for (each spawn point) {
        for (each character on map) {
            if (character at spawn source coordinates)
                ++newplayers;
        }
    }
    
    // 2. Early exit if no one waiting
    if (newplayers == 0)
        return;
    
    // 3. Check capacity (unless forced by admin)
    if (occupants >= block && !force) {
        // Send PACKET_ARENA, PACKET_DROP to all on map
        // (Capacity full message)
        return;
    }
    
    // 4. Prepare warp actions
    for (each spawn point) {
        for (each character) {
            if (character at spawn source) {
                character->next_arena = this;
                character->arena_kills = 0;
                queue_warp(character, spawn destination);
                break; // One player per spawn point
            }
        }
    }
    
    // 5. Execute warps
    for (each queued warp)
        character->Warp(map, x, y);
    
    // 6. Notify clients
    // Send PACKET_ARENA, PACKET_USE with newplayers count
}
```

**Evidence:** `src/arena.cpp:38-118`

#### Packet: Arena Capacity Full

```cpp
PacketBuilder builder(PACKET_ARENA, PACKET_DROP);
// Empty packet, client displays capacity message
```

**Sent to:** All characters on arena map  
**Condition:** `occupants >= block` and not forced

#### Packet: Arena Spawn Notification

```cpp
PacketBuilder builder(PACKET_ARENA, PACKET_USE, 1);
builder.AddChar(newplayers);  // Number of new entrants
```

**Sent to:** All characters on arena map  
**Purpose:** Notify that N players entered

---

### 2. Attack/Elimination Process

**Function:** `Arena::Attack(Character *from, Direction direction)`  
**Trigger:** Called from `Map::Attack()` when `from->arena != NULL`  
**Purpose:** Handle arena-specific attack (elimination)

#### Algorithm

```cpp
void Arena::Attack(Character *from, Direction direction) {
    // 1. Calculate target tile based on attacker direction
    int target_x = from->x;
    int target_y = from->y;
    
    switch (from->direction) {
        case DIRECTION_UP:    target_y -= 1; break;
        case DIRECTION_RIGHT: target_x += 1; break;
        case DIRECTION_DOWN:  target_y += 1; break;
        case DIRECTION_LEFT:  target_x -= 1; break;
    }
    
    // 2. Find victim at target tile
    Character *victim = NULL;
    for (each character on map) {
        if (character->arena == this 
            && character->x == target_x 
            && character->y == target_y) {
            victim = character;
            break;
        }
    }
    
    // 3. If victim found: elimination
    if (victim) {
        ++from->arena_kills;
        
        // Queue warp for victim (to relog point)
        queue_warp(victim, map->relog_x, map->relog_y);
        
        // 4. Send elimination notification
        // PACKET_ARENA, PACKET_SPEC
        // Contains: killer name, victim name, kill count
        
        // 5. Check match end condition
        if (occupants == 2) {
            // Only 2 players, match over
            queue_warp(from, map->relog_x, map->relog_y);
            
            // Send PACKET_ARENA, PACKET_ACCEPT (victory)
        }
        
        // 6. Execute all warps
    }
}
```

**Evidence:** `src/arena.cpp:120-210`

#### Packet: Elimination Notification

```cpp
PacketBuilder builder(PACKET_ARENA, PACKET_SPEC, size);
builder.AddShort(0);              // Unknown
builder.AddByte(255);
builder.AddChar(0);               // Unknown
builder.AddByte(255);
builder.AddInt(from->arena_kills); // Killer's kill count
builder.AddByte(255);
builder.AddBreakString(killer_name);
builder.AddBreakString(victim_name);
```

**Sent to:** All characters on arena map  
**Purpose:** Broadcast elimination

#### Packet: Match Victory

```cpp
PacketBuilder builder(PACKET_ARENA, PACKET_ACCEPT, size);
builder.AddBreakString(winner_name);
builder.AddInt(winner_kills);
builder.AddByte(255);
builder.AddBreakString(winner_name);  // Repeated
builder.AddBreakString(loser_name);
```

**Sent to:** All characters on arena map  
**Condition:** Only 2 occupants remain (1v1 final)  
**Purpose:** Declare match winner

**Evidence:** `src/arena.cpp:180-196`

---

### 3. Occupant Management

**Critical:** Occupant counter tracks active participants

#### Increment (Entry)

**Location:** `Character::Warp()`

```cpp
if (this->next_arena) {
    this->arena = this->next_arena;
    ++this->arena->occupants;      // JOIN
    this->next_arena = 0;
}
```

**Evidence:** `src/character.cpp:1304-1308`

#### Decrement (Exit)

**Location 1:** `Character::Warp()` (leaving arena)

```cpp
if (this->arena) {
    --this->arena->occupants;      // LEAVE
    this->arena = 0;
}
```

**Evidence:** `src/character.cpp:1297-1300`

**Location 2:** `Character::~Character()` (logout/disconnect)

```cpp
if (this->arena) {
    --this->arena->occupants;      // DISCONNECT
}
```

**Evidence:** `src/character.cpp:2229-2231`

---

## Integration Points

### 1. Map System

**Arena per Map:**
- Each `Map` instance has `Arena *arena` (or NULL)
- Loaded via `Map::LoadArena()` during map initialization
- Destroyed in `Map::~Map()` or `Map::LoadArena()` (reload)

**Evidence:** `src/map.cpp:372-448`, `src/map.cpp:741-745`

### 2. Combat System

**Attack Hook:** `Map::Attack()`

```cpp
void Map::Attack(Character *from, Direction direction) {
    // ...
    
    if (from->arena) {
        from->arena->Attack(from, direction);
        return;
    }
    
    // ... normal combat continues
}
```

**Evidence:** `src/map.cpp:1489-1491`

**Effect:** When in arena, attacks bypass normal damage calculation and instantly eliminate.

### 3. Warp System

**State Transfer:** `Character::Warp()`

```cpp
// Before entering new map
if (old_arena) {
    --old_arena->occupants;
    this->arena = 0;
}

// After entering new map
if (this->next_arena) {
    this->arena = this->next_arena;
    ++this->arena->occupants;
    this->next_arena = 0;
}
```

**Evidence:** `src/character.cpp:1296-1308`

### 4. Timer System

**Periodic Spawning:**

```cpp
// In Arena::Arena()
this->spawn_timer = new TimeEvent(arena_spawn, this, time, Timer::FOREVER);
this->map->world->timer.Register(this->spawn_timer);
```

**Callback:**

```cpp
void arena_spawn(void *arena_void) {
    Arena *arena = static_cast<Arena *>(arena_void);
    arena->Spawn();
}
```

**Evidence:** `src/arena.cpp:19-38`

### 5. Admin Commands

**Force Spawn:** `$as` command

```cpp
// src/commands/map.cpp
if (from->map->arena)
    from->map->arena->Spawn(true);
```

**Evidence:** `src/commands/map.cpp:30-32`

**Purpose:** Manually trigger spawn, bypassing capacity limit

---

## PK (Player Kill) Interaction

**Rule:** PK is **automatically disabled** on arena maps.

**Evidence:** `data/arenas.ini:20`

**Implication:**
- Normal PK system does not apply
- Only arena elimination works
- Cannot attack outside arena boundaries (requires `character->arena != NULL`)

---

## Data Flow Diagrams

### Spawn Flow

```
[Timer Event Fires Every N Seconds]
            ↓
    arena_spawn() callback
            ↓
    Arena::Spawn(false)
            ↓
    Count players on spawn tiles
            ↓
    ┌─────────────┐
    │ Any waiting?│ NO → Return
    └─────────────┘
            ↓ YES
    ┌─────────────┐
    │ Capacity OK?│ NO → Send PACKET_ARENA/DROP → Return
    └─────────────┘
            ↓ YES
    For each spawn tile with player:
        Set character->next_arena = this
        Reset character->arena_kills = 0
        Warp to destination
            ↓
    Send PACKET_ARENA/USE to map
            ↓
    Character::Warp() increments occupants
```

### Combat Flow

```
[Player Presses SPACEBAR]
            ↓
    Client sends ATTACK packet
            ↓
    Map::Attack() handler
            ↓
    ┌─────────────┐
    │ In arena?   │ NO → Normal combat
    └─────────────┘
            ↓ YES
    Arena::Attack(from, direction)
            ↓
    Calculate target tile
            ↓
    Find character at target
            ↓
    ┌─────────────┐
    │ Found victim│ NO → Return (miss)
    └─────────────┘
            ↓ YES
    Increment killer->arena_kills
    Warp victim to relog point
    Send PACKET_ARENA/SPEC (elimination)
            ↓
    ┌─────────────┐
    │ Only 2 left?│ NO → Continue match
    └─────────────┘
            ↓ YES
    Warp winner to relog point
    Send PACKET_ARENA/ACCEPT (victory)
```

### Exit Flow

```
[Character Eliminated OR Match Ends]
            ↓
    Arena::Attack() calls Warp(relog_x, relog_y)
            ↓
    Character::Warp()
            ↓
    Decrement old arena->occupants
    Set character->arena = NULL
            ↓
    Enter destination map (usually lobby)
```

---

## Packet Summary

| Packet | Direction | Purpose | Data |
|--------|-----------|---------|------|
| ARENA/DROP | S→C | Capacity full | None |
| ARENA/USE | S→C | Spawn notification | char newplayers |
| ARENA/SPEC | S→C | Elimination | int kills, names |
| ARENA/ACCEPT | S→C | Match victory | winner stats |

**Note:** All packets sent to **all characters on arena map**, not just participants.

---

## Known Limitations

### 1. Fixed Match Rules
- **No scoring system** - Only kill count tracked
- **No time limits** - Match continues until 2 players remain
- **No draws** - Cannot end in tie
- **No rounds** - Single elimination only

### 2. Spawn Mechanics
- **Timer-based only** - Cannot queue manually
- **One player per spawn** - Multiple players on same tile → only first warped
- **No matchmaking** - Players enter in spawn order
- **No team support** - Free-for-all only

### 3. Arena Boundaries
- **No enforcement** - Players can leave arena area on map
- **Entire map is "arena"** - No defined combat zone
- **No spectators** - All on map are participants

### 4. State Persistence
- **No match history** - No database logging
- **No rankings** - No leaderboard
- **No rewards** - No prizes or currency
- **Kill count ephemeral** - Lost on exit

### 5. Configuration
- **Static spawn points** - Cannot change without reload
- **No dynamic arenas** - Must be pre-configured
- **Map-bound** - One arena per map only

---

## Missing Features

The legacy system does **NOT** include:

❌ ELO/rating system  
❌ Matchmaking algorithm  
❌ Match history/logs  
❌ Leaderboards  
❌ Rewards/prizes  
❌ Spectator mode  
❌ Team battles  
❌ Tournament brackets  
❌ Arena shop  
❌ Achievement system  
❌ Statistics tracking  
❌ Replay system  
❌ Custom rulesets  
❌ Dynamic arenas  
❌ Arena UI (uses generic packets)

---

## Usage Example

### Configuration

```ini
# data/arenas.ini
50.enabled = yes
50.time = 60              # Spawn every 60 seconds
50.block = 4              # Max 4 players
50.spawns = 10,10, 5,5, 10,20, 5,15, 20,10, 15,5, 20,20, 15,15
# 4 spawn points:
#   (10,10) → (5,5)
#   (10,20) → (5,15)
#   (20,10) → (15,5)
#   (20,20) → (15,15)
```

### Player Experience

```
1. Player enters map 50 (arena lobby)
2. Player walks to tile (10,10) - spawn point
3. Player waits up to 60 seconds
4. Timer fires, Arena::Spawn() called
5. Player warped to (5,5) in combat area
6. character->arena set, character->arena_kills = 0
7. Player attacks opponents by pressing spacebar
8. Each kill increments arena_kills
9. When only 2 players remain:
   - Winner declared
   - Both warped to relog point
10. Player back in lobby, arena = NULL
```

---

## Related Files

| File | Purpose |
|------|---------|
| `src/arena.hpp` | Arena class definition |
| `src/arena.cpp` | Arena implementation |
| `src/character.hpp` | Character arena state |
| `src/character.cpp` | Warp integration |
| `src/map.hpp` | Map arena pointer |
| `src/map.cpp` | Arena loading, attack hook |
| `src/commands/map.cpp` | Admin force spawn |
| `data/arenas.ini` | Arena configuration |
| `config/files.ini` | Override arena config file |

---

## Next Steps for Modernization

See `LEGACY-ARENA-BUGS.md` for critical issues that must be fixed before extending this system.

Key areas for improvement:
1. **Matchmaking** - Replace timer with queue system
2. **State Management** - Fix reentrancy issues and state desyncs
3. **Validation** - Add bounds checking and input validation
4. **Persistence** - Add database logging
5. **UI** - Use modern window systems (Quest, Board, Message)
6. **Features** - Add ELO, leaderboards, rewards

---

**Summary:** The legacy arena system is a minimal, timer-based PvP framework with instant-elimination combat. It provides basic functionality but lacks modern features and contains critical bugs. Recommended approach: Document thoroughly, fix critical bugs, then build new system on top using proven patterns from other ArenaServ systems (quests, boards, shops).

