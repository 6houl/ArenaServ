# Arena Test Seams Analysis

**Date:** 2026-09-23  
**Purpose:** Document Arena dependencies and testability for ISSUE-001

---

## Arena Class Structure

```cpp
class Arena
{
public:
    int occupants;          // Current occupant count
    int time;               // Spawn timer interval
    int block;              // Capacity/block threshold
    
    TimeEvent *spawn_timer; // Timer for periodic spawns
    Map *map;               // Parent map reference
    std::vector<Arena_Spawn *> spawns; // Spawn point configurations
};
```

---

## Direct Dependencies

### 1. Map (REQUIRED)
- Arena constructor requires `Map*`
- Arena references `map->characters` (iterate all characters on map)
- Arena uses `map->world->timer` to register spawn timer
- Arena uses `map->id`, `map->relog_x`, `map->relog_y` for warps
- **Testability:** Must provide Map instance or mock

### 2. Timer (REQUIRED)
- Arena creates `TimeEvent` (spawn_timer) in constructor
- Timer registered with `map->world->timer`
- Timer callback `arena_spawn()` invokes `Arena::Spawn()`
- **Testability:** Must provide Timer infrastructure or mock

### 3. Character (REQUIRED for Spawn/Attack)
- `Arena::Spawn()` iterates `map->characters`
- Checks character position (`character->x`, `character->y`)
- Sets `character->next_arena`, `character->arena_kills`
- Calls `character->Warp()` for spawn/attack actions
- Calls `character->Send()` for packets
- **Testability:** Must provide Character instances or mocks

### 4. World (INDIRECT via Map)
- Accessed via `map->world->timer`
- Required for timer registration
- **Testability:** Provided by Map dependency

### 5. Packet/EOClient (INDIRECT via Character)
- `PacketBuilder` used to construct packets
- Sent via `character->Send()`
- **Testability:** Character mock must handle Send()

---

## Configuration Dependencies

### Arena Configuration Loading
- Loaded by `Map::LoadArena()` (src/map.cpp:372)
- Reads from config file (`data/arenas.ini`)
- Format:
  ```ini
  mapid.enabled = yes
  mapid.time = 30
  mapid.block = 3
  mapid.spawns = sx1,sy1,dx1,dy1, sx2,sy2,dx2,dy2
  ```
- **Testability:** Can provide test config or construct Arena directly

---

## State Dependencies

### Match State (via Character)
- `character->arena` indicates current arena
- `character->next_arena` indicates pending arena entry
- `character->arena_kills` tracks kills in arena
- **Testability:** Character mock must support these fields

### Occupant Tracking
- `arena->occupants` managed by Character class (NOT Arena)
- Incremented/decremented in `src/character.cpp` (Bug #2 location)
- **Testability:** Must control occupant count for state tests

---

## Test Seam Options

### Option 1: Direct Construction (SIMPLEST)
```cpp
// Requires: real Map, World, Timer
Map* testMap = /* create or mock */;
Arena arena(testMap, 30, 5);
```
**Pros:** Tests real Arena code  
**Cons:** Requires full World/Map/Timer infrastructure

### Option 2: Existing Test Helpers
```cpp
// Use CreateMockDatabase(), CreateConfigWithTestDefaults() from setup.hpp
// Construct World with test config
// Create test Map
// Construct Arena normally
```
**Pros:** Leverage existing test infrastructure  
**Cons:** Still requires World/Map setup

### Option 3: Minimal Test Fixture
```cpp
class ArenaTestFixture : public ::testing::Test
{
protected:
    std::unique_ptr<World> testWorld;
    Map* testMap;
    Arena* testArena;
    
    void SetUp() override {
        // Create minimal World with test config
        // Create test Map
        // Create Arena
    }
};
```
**Pros:** Encapsulates setup complexity  
**Cons:** Requires World infrastructure

### Option 4: Behavior-Neutral Test Seam (LAST RESORT)
Add Arena constructor overload:
```cpp
Arena(Map *map, int time, int block, bool register_timer = true);
```
**Pros:** Allows testing without timer  
**Cons:** Modifies production code

---

## Recommended Approach for ISSUE-001

**Strategy:** Option 3 (Minimal Test Fixture) using existing test helpers

**Rationale:**
- Leverages existing `CreateMockDatabase()`, `CreateConfigWithTestDefaults()`
- Tests real Arena behavior (not mocked away)
- Encapsulates complexity in fixture
- No production code changes required

**Implementation Plan:**
1. Create `src/test/arena_test.cpp`
2. Use existing `MockDatabase` from test helpers
3. Create minimal World with test config
4. Create test Map on minimal World
5. Construct Arena normally
6. Provide mock or minimal Character instances for state tests

---

## Character Test Support

**Challenge:** Arena requires Character instances for Spawn/Attack tests.

**Options:**
1. Use existing test helper mocks (`MockCharacter`?) — check if exists
2. Create minimal real Character instances with test data
3. Create Arena-specific Character stub

**Decision:** Check for existing Character test support first.

---

## Timer Test Support

**Challenge:** Arena spawn timer fires periodically (`arena_spawn()` callback).

**Options:**
1. Use real Timer infrastructure (tests timer integration)
2. Mock timer (test Arena logic without timing)
3. Trigger `Arena::Spawn()` directly (bypass timer for baseline tests)

**Decision:** Trigger `Spawn()` directly for baseline tests. Timer integration can be separate test.

---

## Configuration Test Support

**Challenge:** Arena configuration loaded from `arenas.ini` by Map.

**Options:**
1. Create test `arenas.ini` files
2. Construct Arena directly with test parameters (bypass config loading)
3. Test config loading separately from Arena logic

**Decision:** Construct Arena directly for unit tests. Config loading tested separately.

---

## Test Fixture Minimal Requirements

To support baseline Arena tests, fixture must provide:

**Required:**
- `World*` instance (can be minimal)
- `Map*` instance (minimal, on test World)
- `Arena*` instance (constructed on test Map)

**For Spawn/Attack Tests:**
- `Character*` instances (minimal or mocked)
- Ability to place characters at specific (x,y) positions
- Ability to verify character state (`arena`, `arena_kills`, warped)

**NOT Required (Initially):**
- Full database
- Full packet handling
- Client connections
- Multiple concurrent arenas
- Real timer callbacks (use `Spawn()` directly)

---

## Baseline Test Coverage Plan

With minimal fixture, implement:

1. **ARENA-CONFIG-001:** Arena construction succeeds with valid parameters
2. **ARENA-ENTRY-001:** `Spawn()` can be called (baseline smoke)
3. **ARENA-ENTRY-002:** `occupants` field accessible (state check)
4. **ARENA-STATE-001:** Spawn when occupants < block (normal flow)
5. **ARENA-STATE-002:** Spawn when occupants >= block (capacity check)
6. **ARENA-MATCH-001:** `Attack()` can be called without crash

**Bug Reproduction (if feasible without production changes):**
- Bug #3: Construct Arena_Spawn with out-of-bounds coordinates → verify bounds check (currently missing)
- Bug #2: Manually manipulate occupants → verify desync scenario
- Bug #5: Set arena_kills to INT_MAX → verify overflow handling

---

## Production Code Changes Required

**NONE** for baseline tests.

If test seams absolutely required:
- Document why needed
- Minimize impact
- Prove behavior unchanged

**Current Assessment:** Can implement baseline tests WITHOUT production changes.

---

## Next Steps

1. Check if `MockCharacter` or similar exists in test helpers
2. Create `ArenaTestFixture` in `src/test/arena_test.cpp`
3. Implement baseline Arena tests
4. Update test matrix with implementation status

