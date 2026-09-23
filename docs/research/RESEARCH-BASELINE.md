# ArenaServ Research Baseline

**Established:** 2026-09-22 (Phase 0.5)  
**Purpose:** Record verified facts as ground truth for future development

---

## Repository Information

### Project Identity

**Name:** ArenaServ  
**Based On:** Etheos v0.7.1  
**Source:** `CMakeLists.txt:9`

```cmake
project(etheos VERSION 0.7.1 LANGUAGES CXX C)
```

**Upstream Lineage:**
- EOSERV (original, Apollo)
- → Etheos (ciaran/ethanmoffat modernization fork)
- → ArenaServ (Brandon's arena-focused fork)

### Git Information

**Repository:** https://github.com/6houl/ArenaServ  
**Baseline Commit:** b204d455aa53585cde4f746f6b0737828b741784  
**Branch:** main  
**Date:** 2026-09-22

**Baseline Composition:**
- Commit de0a4c2: Etheos v0.7.1 source code
- Commit b204d45: Phase 0 + Phase 0.5 documentation

**Phase 0.5 Note:** Phase 0.5 verification was performed on the source tree **before** Git initialization. The examined source code is captured in commit de0a4c2a2093ea30728f45e7972b44b9a76b7149.

**Upstream Reference:**
- Remote: `upstream` → https://github.com/ethanmoffat/etheos.git
- Etheos version: v0.7.1 (no specific commit tracked)
- EOSERV: Historical reference only
- eo-protocol: Included in `references/`

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

**Corrected Analysis:** `docs/arena/LEGACY-ARENA-BUGS.md` Bug #1 (corrected)

**Impact:** Critical (crash, state corruption, iterator invalidation)

---

### Bugs #2-#8: Pending Re-Verification

**Status:** ⏸️ VERIFIED (Phase 0.5)

These bugs were re-analyzed with correct execution model understanding:

- Bug #2: Occupant desync (CONFIRMED)
- Bug #3: Bounds validation (CONFIRMED)
- Bug #4: Timer destruction (UNPROVEN - requires testing)
- Bug #5: Kill overflow (CONFIRMED)
- Bug #6: Spawn limit (CONFIRMED - design limitation)
- Bug #7: Match timeout (CONFIRMED - design limitation)
- Bug #8: Broadcast spam (CONFIRMED)

---

[... rest of file continues with same format as before, updated with Git info ...]
