# ArenaServ Research Baseline

**Established:** 2026-09-22 (Phase 0.5)  
**Purpose:** Record verified facts as ground truth for future development

---

## Repository Information

### Project Identity

**Name:** Etheos  
**Version:** 0.7.1  
**Source:** `CMakeLists.txt:9`

```cmake
project(etheos VERSION 0.7.1 LANGUAGES CXX C)
```

**Upstream Lineage:**
- EOSERV (original, Apollo)
- → Etheos (ciaran modernization fork)
- → ArenaServ (Brandon's arena-focused fork)

**Git Status:** Not a git repository  
**Commit Hash:** N/A (no .git directory)

---

## Verified Execution Model

### GROUND TRUTH: Single-Threaded Architecture

**Status:** ✅ VERIFIED (Phase 0.5)

Etheos uses a **single-threaded event loop**. There is NO concurrent execution, NO thread pool, NO parallel processing.

#### Evidence

**Main Loop:**
```cpp
// src/main.cpp:461-468
while (eoserv_running)
{
    server->Tick();  // ALL work happens here, sequentially
}
```

**Timer System:**
```cpp
// src/timer.cpp:182-257
void Timer::Tick()
{
    impl->lock();  // Acquire mutex
    
    // ... iterate timers
    
    impl->unlock();  // Release BEFORE callback
    timer->callback(timer->param);  // Execute unlocked
    impl->lock();  // Reacquire AFTER callback
    
    impl->unlock();  // Final release
}
```

#### Implications

1. ❌ NO "race conditions" possible (requires concurrent execution)
2. ✅ Reentrancy IS possible (nested calls during callbacks)
3. ❌ ThreadSanitizer NOT needed (no threads)
4. ✅ AddressSanitizer IS useful (memory errors still possible)

**Reference:** `docs/architecture/EXECUTION-MODEL.md`

---

## Verified Bug Classifications

### Bug #1: Reentrancy Issue (NOT Race Condition)

**Status:** ✅ VERIFIED and CORRECTED (Phase 0.5)

**Original Phase 0 Claim:** "Race condition in simultaneous attacks"

**Verified Reality:**
- NOT a race condition (no concurrent execution)
- IS a reentrancy issue (callbacks during attack processing)
- Queued actions operate on stale state
- Timer/packet callbacks can trigger during send operations

**Evidence:**
- src/arena.cpp:120-210 - Arena::Attack() action queue
- src/timer.cpp:182-257 - Timer unlocks during callback
- src/main.cpp:461-468 - Single-threaded main loop

**Corrected Analysis:** `docs/architecture/BUG1-CORRECTED.txt`

**Impact:** Critical (crash, state corruption, iterator invalidation)

---

### Bugs #2-#8: Pending Re-Verification

**Status:** ⏸️ PENDING (Phase 0.5 in progress)

These bugs require re-analysis with correct execution model understanding:

- Bug #2: Occupant desync (likely still valid)
- Bug #3: Bounds validation (likely still valid, input validation)
- Bug #4: Timer UAF (requires deep analysis with reentrancy model)
- Bug #5: Kill overflow (likely still valid, data type issue)
- Bug #6: Spawn limit (likely still valid, logic flaw)
- Bug #7: Match timeout (likely still valid, design flaw)
- Bug #8: Broadcast spam (likely still valid, UX issue)

---

## File System Structure

### Source Code Organization

```
src/
├── main.cpp - Main loop, server startup
├── timer.cpp/hpp - Timer system with reentrancy
├── arena.cpp/hpp - Legacy arena implementation
├── character.cpp/hpp - Character state and actions
├── map.cpp/hpp - Map loading and management
├── eoserver.cpp/hpp - Server tick and packet processing
└── ... (other files)
```

### Configuration Files

```
config/
├── server.ini - Server config
├── database.ini - Database selection
├── database/
│   ├── mysql.ini
│   ├── sqlite.ini
│   └── sqlserver.ini
└── ... (other configs)
```

### Data Files

```
data/
├── arenas.ini - Arena spawn definitions
├── maps/ - EMF map files
├── pub/ - Client data files (EIF, ENF, ESF, ECF)
└── ... (other data)
```

---

## Build Environment

### Verified Build System

**Build Tool:** CMake 3.15+  
**Language:** C++11 minimum  
**Compiler Support:**
- GCC 7.3+ (Linux)
- MSVC 2019+ (Windows)
- Clang (untested)

### Dependencies

**Required:**
- MariaDB C Connector (if using MySQL backend)
- SQLite3 (if using SQLite backend)

**Optional:**
- Google Test (for unit tests)
- bcrypt (downloaded by CMake if not found)

### Build Commands

**Windows (PowerShell):**
```powershell
.\build-windows.ps1
```

**Linux (Bash):**
```bash
./build-linux.sh
```

**Reference:** `docs/BUILD-ENVIRONMENT.md`

---

## Protocol Information

### Packet Families

**Total Families:** 51  
**Arena-Relevant:** ~20  
**Critical for Arena:** 16

**High Priority:**
- PACKET_WARP - Character teleportation
- PACKET_ATTACK - Combat actions
- PACKET_MESSAGE - Chat and announcements
- PACKET_QUEST - Dialog/quest system
- PACKET_BOARD - Bulletin board (for rankings)

**Legacy (Avoid):**
- PACKET_ARENA - Legacy arena protocol (deprecated)

**Reference:** `docs/protocol/PACKET-CATALOG.md`

---

## Data File Formats

### EO Data Files

**Count:** 5 file types  
**Status:** Fully documented (Phase 0)

| Format | Purpose | Structure | Verified |
|--------|---------|-----------|----------|
| EIF | Items | Binary, fixed records | ✅ Phase 0 |
| ENF | NPCs | Binary, fixed records | ✅ Phase 0 |
| ESF | Spells | Binary, fixed records | ✅ Phase 0 |
| ECF | Classes | Binary, fixed records | ✅ Phase 0 |
| EMF | Maps | Binary, tiled | ✅ Phase 0 |

### Key Insight

Server can create custom items/NPCs that work with vanilla client by reusing existing type IDs.

**Reference:** `docs/data-files/DATA-FILE-FORMATS.md`

---

## Client Capabilities

### Vanilla Client v28

**Status:** Documented (Phase 0), not yet verified (Phase 0.5)

**Windows Documented:**
- Main game screen
- Character creation/selection
- Inventory, spells, stats
- Quest dialog system
- Board system
- Guild UI
- Jukebox

**Arena Relevance:**
- Quest dialogs can show arena info
- Board can show rankings
- Main screen shows arena combat

**Reference:** `docs/client/CLIENT-CAPABILITY-MATRIX.md`

---

## Unknown Behaviors

### Research Questions

**Total Questions:** 29  
**Categories:** 7  
**Priority Levels:** P0 (critical) to P4 (nice-to-have)

**Critical P0 Questions (4):**
1. Quest dialog window limits
2. Board entry limits
3. Packet size limits
4. File sync behavior edge cases

**Reference:** `docs/research/UNKNOWN-BEHAVIOR.md`

---

## Documentation Inventory

### Phase 0 Documents (14 files)

1. README.md
2. PROJECT-OVERVIEW.md
3. AI-DEVELOPMENT-RULES.md
4. UPSTREAM-LINEAGE.md
5. BUILD-ENVIRONMENT.md
6. ROADMAP.md
7. GITHUB-WORKFLOW.md
8. PHASE-0-REPORT.md
9. CLIENT-CAPABILITY-MATRIX.md
10. WINDOW-CAPABILITIES.md
11. PACKET-CATALOG.md
12. DATA-FILE-FORMATS.md
13. LEGACY-ARENA-ARCHITECTURE.md
14. LEGACY-ARENA-BUGS.md
15. UNKNOWN-BEHAVIOR.md

**Total Lines:** ~40,000

### Phase 0.5 Documents (3 files)

16. EXECUTION-MODEL.md (NEW)
17. BUG1-CORRECTED.txt (NEW)
18. PHASE-0.5-REPORT.md (NEW)
19. RESEARCH-BASELINE.md (NEW, this document)

**Total Lines:** ~5,000 (Phase 0.5)

---

## Verification Principles

### Established Guidelines

1. **Source code outranks documentation** - If conflict, code wins
2. **Terminology must be precise** - Technical accuracy required
3. **Evidence-based claims only** - Every claim needs source location
4. **Trace execution paths** - Don't assume, verify
5. **Adversarial verification** - Challenge Phase 0 claims

### Verification Status

| Phase 0 Area | Verification Status |
|--------------|---------------------|
| Execution model | ✅ VERIFIED (corrected) |
| Bug #1 | ✅ VERIFIED (corrected) |
| Bugs #2-#8 | ⏸️ PENDING |
| Client UI | ⏸️ PENDING |
| Packet catalog | ⏸️ PENDING |
| Data files | ⏸️ PENDING |
| Arena lifecycle | ⏸️ PENDING |
| Database schema | ⏸️ PENDING |

---

## Key Discoveries

### Discovery #1: Single-Threaded Execution

**Date:** 2026-09-22 (Phase 0.5)

**Finding:** Etheos uses single-threaded main loop, NOT multi-threaded

**Impact:** Invalidates all "race condition" terminology in Phase 0

**Evidence:**
- src/main.cpp:461-468 - Single while loop
- No pthread or std::thread found
- Timer callbacks execute sequentially

**Documents Updated:**
- NEW: docs/architecture/EXECUTION-MODEL.md
- PENDING: docs/arena/LEGACY-ARENA-BUGS.md

---

### Discovery #2: Timer Reentrancy Window

**Date:** 2026-09-22 (Phase 0.5)

**Finding:** Timer::Tick() unlocks mutex during callback execution

**Impact:** Enables reentrancy but NOT concurrency

**Evidence:**
- src/timer.cpp:182-257 - Explicit unlock before callback

**Mechanism:**
```
impl->lock()
impl->unlock()      ← Release lock
callback()          ← Can call Timer::Register/Unregister
impl->lock()        ← Reacquire lock
```

**Implication:** Callbacks can modify timer list, register new timers, trigger warps, process packets

---

## Testing Implications

### What Works

✅ **AddressSanitizer** - Memory errors  
✅ **Valgrind** - Memory leaks, UAF  
✅ **Unit tests** - Component isolation  
✅ **Reentrancy tests** - Trigger callbacks during operations  

### What Doesn't Work

❌ **ThreadSanitizer** - No threads to analyze  
❌ **Concurrent load tests** - Single-threaded bottleneck  
❌ **Race condition fuzzing** - No races possible  

### Recommended Strategy

1. Focus on reentrancy testing
2. Trigger timer callbacks during critical operations
3. Use AddressSanitizer for memory errors
4. Manually trigger nested call scenarios
5. Test with slow network (delays between packets)

---

## Future Work

### Phase 0.5 Remaining Tasks

1. ⏸️ Complete bug re-verification (Bugs #2-#8)
2. ⏸️ Verify client UI claims
3. ⏸️ Create experiment register
4. ⏸️ Verify packet documentation
5. ⏸️ Verify data file claims
6. ⏸️ Map arena state machine
7. ⏸️ Trace arena lifecycle
8. ⏸️ Verify database schema
9. ⏸️ Update UNKNOWN-BEHAVIOR.md
10. ⏸️ Create evidence ledger
11. ⏸️ Final verification report

### Phase 1 Prerequisites

Before Phase 1 implementation can begin:

1. ✅ Execution model verified
2. ⏸️ All 8 bugs verified and prioritized
3. ⏸️ Client capabilities confirmed
4. ⏸️ Packet protocol validated
5. ⏸️ Test strategy established

---

## References

### Key Source Files

- src/main.cpp - Main loop and server startup
- src/timer.cpp - Timer system (reentrancy window)
- src/arena.cpp - Legacy arena implementation
- src/character.cpp - Character state management
- CMakeLists.txt - Project version and build config

### Key Documentation

- docs/architecture/EXECUTION-MODEL.md - Execution model (Phase 0.5)
- docs/arena/LEGACY-ARENA-BUGS.md - Bug catalog (Phase 0, needs update)
- docs/protocol/PACKET-CATALOG.md - Packet reference (Phase 0)
- docs/data-files/DATA-FILE-FORMATS.md - Data file formats (Phase 0)

---

## Change Log

### 2026-09-22 (Phase 0.5 Start)

- ✅ Established research baseline
- ✅ Verified single-threaded execution model
- ✅ Corrected Bug #1 classification
- ✅ Created EXECUTION-MODEL.md
- ✅ Created PHASE-0.5-REPORT.md
- ✅ Created RESEARCH-BASELINE.md (this document)

---

**Status:** Phase 0.5 in progress - Ground truth established for execution model
