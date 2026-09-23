# ArenaServ Evidence Ledger

**Purpose:** Authoritative record of verified architectural facts  
**Status:** Phase 0.5 Complete  
**Date:** 2026-09-22

## Critical Architectural Facts

### FACT-001: Single-Threaded Execution

**Claim:** Etheos v0.7.1 uses single-threaded main loop  
**Evidence:** `src/main.cpp:461-468`  
**Status:** ✅ VERIFIED  
**Date:** 2026-09-22

```cpp
while (eoserv_running)
{
    server->Tick();  // Single thread processes all work
}
```

**Implication:** No concurrent execution. Issues are reentrancy, not races.

**References:** `docs/architecture/EXECUTION-MODEL.md`

---

### FACT-002: Timer Reentrancy Window

**Claim:** Timer::Tick() unlocks mutex during callback execution  
**Evidence:** `src/timer.cpp:199, 217, 253`  
**Status:** ✅ VERIFIED  
**Date:** 2026-09-22

```cpp
impl->unlock();  // Release before callback
timer->callback(timer->param);
impl->lock();   // Reacquire after
```

**Implication:** Callbacks can register/unregister timers, trigger warps, process packets.

---

### FACT-003: Character::Warp is Synchronous

**Claim:** Warp executes immediately, not queued  
**Evidence:** `src/character.cpp:1296-1308`  
**Status:** ✅ VERIFIED  
**Date:** 2026-09-22

```cpp
if (this->arena) {
    --this->arena->occupants;  // Immediate
    this->arena = 0;            // Immediate
}
```

**Implication:** State updates happen inline, not deferred.

---

### FACT-004: Arena Has Zero Persistence

**Claim:** Legacy arena stores no database state  
**Evidence:** `install.sql` (no arena tables), `src/character.hpp:205-207` (runtime only)  
**Status:** ✅ VERIFIED  
**Date:** 2026-09-22

No `CREATE TABLE` for arena data in:
- `install.sql`
- `install_sqlserver.sql`
- `upgrade/*.sql`

All arena state is runtime pointers.

**Implication:** ArenaServ requires entirely new schema.

---

### FACT-005: Map Owns Arena

**Claim:** Map directly owns Arena (raw pointer)  
**Evidence:** `src/map.hpp:232` (`Arena *arena`), `src/map.cpp:386,745` (`delete this->arena`)  
**Status:** ✅ VERIFIED  
**Date:** 2026-09-22

```cpp
class Map {
    Arena *arena;  // Raw pointer, Map owns
};

Map::LoadArena() {
    delete this->arena;  // Direct deletion
    this->arena = new Arena(...);
}
```

**Implication:** Arena lifetime tied to Map lifetime. Map reload destroys arena.

---

### FACT-006: TimeEvent Self-Unregisters

**Claim:** ~TimeEvent() calls Unregister if manager set  
**Evidence:** `src/timer.cpp:314-318`  
**Status:** ✅ VERIFIED  
**Date:** 2026-09-22

```cpp
~TimeEvent() {
    if (this->manager != 0) {
        this->manager->Unregister(this);
    }
}
```

**Implication:** Deleting TimeEvent safely removes from Timer. Prevents dangling pointers.

---

### FACT-007: Board System Implemented

**Claim:** Board packets functional in ArenaServ  
**Evidence:** `src/handlers/Board.cpp`, `src/character.cpp:1403`  
**Status:** ✅ VERIFIED  
**Date:** 2026-09-22

Handlers registered:
- `PACKET_BOARD/PACKET_OPEN` - Open board
- `PACKET_BOARD/PACKET_PLAYER` - Read post
- `PACKET_BOARD/PACKET_CREATE` - Create post
- `PACKET_BOARD/PACKET_REMOVE` - Delete post

**Implication:** Board can be used for rankings without client modification.

---

### FACT-008: Quest Dialog Implemented

**Claim:** Quest dialog packets functional  
**Evidence:** `src/handlers/Quest.cpp:61`  
**Status:** ✅ VERIFIED  
**Date:** 2026-09-22

```cpp
PacketBuilder reply(PACKET_QUEST, PACKET_DIALOG, 10);
reply.AddChar(static_cast<unsigned char>(dialogs.size()));
reply.AddShort(vendor_id);
```

**Implication:** Quest dialogs can show arena menus without client modification.

---

### FACT-009: Spawn Coordinates Unvalidated

**Claim:** No bounds checking on spawn coordinates  
**Evidence:** `src/map.cpp:415-427` (no validation)  
**Status:** ✅ VERIFIED  
**Date:** 2026-09-22

```cpp
s->sx = util::to_int(spawn);  // No check vs map width
s->sy = util::to_int(spawn);  // No check vs map height
```

**Implication:** Invalid config can cause buffer overflow.

---

### FACT-010: Kill Count Uses Signed Char

**Claim:** arena_kills is char (8-bit signed)  
**Evidence:** `src/character.hpp:207`  
**Status:** ✅ VERIFIED  
**Date:** 2026-09-22

```cpp
char arena_kills;  // -128 to 127
```

**Implication:** Overflows at 128 kills → -128.

---

### FACT-011: Project Version

**Claim:** Repository is Etheos v0.7.1  
**Evidence:** `CMakeLists.txt:9`  
**Status:** ✅ VERIFIED  
**Date:** 2026-09-22

```cmake
project(etheos VERSION 0.7.1 LANGUAGES CXX C)
```

**Repository:** Not under git control (no `.git` directory)

---

### FACT-012: Database Backend Support

**Claim:** Supports MySQL, SQLite, SQL Server  
**Evidence:** `config/database/*.ini`, `install.sql`, `install_sqlserver.sql`  
**Status:** ✅ VERIFIED  
**Date:** 2026-09-22

Three backend configs present, two install scripts.

**Implication:** ArenaServ schema must support all three.

---

## Rejected Claims

### REJECTED-001: Bug #1 is a Race Condition

**Original Claim:** Simultaneous attacks cause race condition  
**Evidence:** Execution model is single-threaded  
**Status:** ❌ REJECTED  
**Date:** 2026-09-22

**Corrected:** Bug #1 is a reentrancy issue with deferred action queue.

**Reference:** `docs/arena/LEGACY-ARENA-BUGS.md` Bug #1 (corrected)

---

### REJECTED-002: Bug #4 is Use-After-Free of Arena

**Original Claim:** Arena accessed after destruction  
**Evidence:** ~TimeEvent() calls Unregister, preventing timer access  
**Status:** ❌ UNPROVEN  
**Date:** 2026-09-22

**Corrected:** Potential double-free of timer, not UAF of arena. Requires testing.

**Reference:** `docs/arena/LEGACY-ARENA-BUGS.md` Bug #4 (analysis)

---

## Confidence Levels

**✅ VERIFIED:** Source code proven, execution traced  
**🔶 PROBABLE:** Strong evidence, not yet tested  
**❓ EXPERIMENTAL:** Requires client testing  
**❌ REJECTED:** Contradicted by evidence  
**⚠️ UNKNOWN:** Insufficient evidence

---

## Update Protocol

When new facts are verified:
1. Add FACT-NNN entry with evidence
2. Link to source files with line numbers
3. State implications for design
4. Update affected documentation

When claims are rejected:
1. Add REJECTED-NNN entry
2. Explain contradiction
3. Provide corrected analysis
4. Update affected documents

---

**Evidence Standard:** Every fact must cite source file, line number, or config location.

**No fact based solely on Phase 0 documents** - all traced to primary sources.
