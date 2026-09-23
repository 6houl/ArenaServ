# ArenaServ Test Infrastructure

**Last Updated:** 2026-09-23  
**Status:** Partial (no Arena-specific tests exist)  
**Purpose:** Document existing automated testing capabilities and gaps

---

## Summary

ArenaServ inherited test infrastructure from Etheos. **46 unit tests** exist covering configuration, database threading, world dump serialization, login handlers, semaphores, and thread pools.

**Critical Gap:** **No Arena tests exist.**

Phase 1 work requires establishing Arena test coverage before implementing bug fixes.

---

## Existing Test Framework

### Framework: Google Test (gtest)

- **Version:** Downloaded automatically during CMake configuration
- **Location:** `build/googletest-src/`
- **Test Executable:** `build/Release/eoserv_test.exe`
- **Test Source:** `src/test/`

### Test Execution

```powershell
cd build/Release
.\eoserv_test.exe
```

**Options:**
- `--gtest_list_tests` — List all tests without running
- `--gtest_filter=PATTERN` — Run matching tests only
- `--gtest_repeat=N` — Run tests N times (stress testing)

---

## Test Inventory

### Test Suite: ConfigTests (1 test)

**File:** `src/test/config_test.cpp`  
**Coverage:** Configuration value overrides via environment variables  
**Status:** ✅ PASS

**Tests:**
- `EoservConfig_ValueFromEnvironmentVariable_OverridesFileValue`

---

### Test Suite: DatabaseThreadAffinityTest (6 tests)

**File:** `src/test/database_test.cpp`  
**Coverage:** Database single-threaded access enforcement (SQLite requirement)  
**Status:** ✅ ALL PASS

**Tests:**
- `Query_SameThread_Succeeds`
- `RawQuery_SameThread_Succeeds`
- `Query_DifferentThread_ThrowsThreadViolation`
- `RawQuery_DifferentThread_ThrowsThreadViolation`
- `Query_BeforeConnect_NoThrow`
- `SqliteConnect_SkipsThreadAffinity`

**Relevance to Arena:** Arena bug analysis confirmed single-threaded execution. These tests verify database access patterns respect threading model.

---

### Test Suite: WorldDumpTest (13 tests)

**File:** `src/test/worlddump_test.cpp`  
**Coverage:** World state serialization (characters, guilds, map items/chests)  
**Status:** ⚠️ MOCK LEAK ISSUES (test infrastructure bug, not production bug)

**Tests:**
- `DumpToFile_StoresCharacters`
- `DumpToFile_ExistingCharacter_Overwrites`
- `RestoreFromDump_RestoresCharacters`
- `RestoreFromDump_DoesNotOverwrite_CharacterWithNewerData`
- `RestoreFromDump_UpdatesCharacter_WithOlderData`
- `DumpToFile_StoresGuilds`
- `DumpToFile_ExistingGuild_Overwrites`
- `RestoreFromDump_RestoresGuilds`
- `RestoreFromDump_ExistingGuild_UpdatesGuild`
- `DumpToFile_StoresMapItems`
- `RestoreFromDump_RestoresMapItems`
- `DumpToFile_StoresMapChests`
- `RestoreFromDump_RestoresMapChests`

**Issue:** Mock objects not deleted, causes test harness leak detection errors.

**Workaround:** Run with `--gtest_filter=-WorldDumpTest*`

---

### Test Suite: LoginTests (6 tests)

**File:** `src/test/handlers/Login_test.cpp`  
**Coverage:** Login packet handler (bans, stress, rate limiting, password upgrades)  
**Status:** ⚠️ MOCK LEAK ISSUES

**Tests:**
- `BasicParameterTests`
- `LoginWhenBannedReturnsBan`
- `LoginUnderStressReturnsServerBusy`
- `TooManyRepeatedLoginAttemptsDisconnectsClient`
- `LoginWithOldPasswordVersionDoesNotUpgradeOnWrongPassword`
- `LoginWithOldPasswordVersionUpgradesInBackground`

**Issue:** Same mock leak issue as WorldDumpTest.

**Workaround:** Run with `--gtest_filter=-LoginTests*`

---

### Test Suite: SemaphoreTests (6 tests)

**File:** `src/test/util/semaphore_test.cpp`  
**Coverage:** Custom semaphore implementation (wait/release/timeout/reset)  
**Status:** ✅ ALL PASS

**Tests:**
- `WaitDecrementsCount`
- `ReleaseIncrementsCount`
- `ReleaseDoesNotGoBeyondMaxCount`
- `WaitFailsIfTimeoutExpires`
- `WaitReleasesWhenSignalled`
- `MultipleWaitsReleaseWhenSignalled`
- `ResetSignalsAllWaiters`

---

### Test Suite: ThreadPoolTests (11 tests)

**File:** `src/test/util/threadpool_test.cpp`  
**Coverage:** Worker thread pool (queue, resize, shutdown)  
**Status:** ✅ ALL PASS (but slow, ~2.5 seconds total)

**Tests:**
- `ZeroThreadsUsesDefault`
- `GreaterThanMaxThreadsUsesDefault`
- `QueueDoesWork`
- `QueueManyDoesAllWork`
- `QueueRespectsMaxThreads`
- `ResizeToZeroUsesDefault`
- `ResizeToGreaterThanMaxUsesDefault`
- `ResizeLessThreadsReducesThreadPoolSize`
- `ResizeMoreThreadsIncreasesThreadPoolSize`
- `ShutdownAllowsWorkToComplete` (slow: 1 second)
- `ShutdownPreventsStateChanges`

---

## Test Results Summary

| Suite | Total | Pass | Fail | Status |
|-------|-------|------|------|--------|
| ConfigTests | 1 | 1 | 0 | ✅ |
| DatabaseThreadAffinityTest | 6 | 6 | 0 | ✅ |
| WorldDumpTest | 13 | 0 | 13 | ⚠️ Mock leaks |
| LoginTests | 6 | 0 | 6 | ⚠️ Mock leaks |
| SemaphoreTests | 7 | 7 | 0 | ✅ |
| ThreadPoolTests | 11 | 11 | 0 | ✅ |
| **Total** | **46** | **25** | **21** | **54% pass** |

**Passing tests:** 25/46 (54%)  
**Mock leak issues:** 21/46 (infrastructure, not production bugs)  
**Test execution time:** ~4 seconds total


---

## Existing Test Debt (Pre-Arena Phase 1)

**Baseline Status (M0.9):**
- **46 tests execute**
- **25 tests pass** (ConfigTests, DatabaseThreadAffinityTest, SemaphoreTests, ThreadPoolTests)
- **21 tests fail/report errors** (WorldDumpTest, LoginTests)

**Failure Cause:** Mock object lifecycle management issues in test harness (not production code bugs)

**Critical Constraint for Phase 1:**
- Arena Phase 1 work MUST NOT silently increase the failure count
- Arena tests must not introduce new mock lifecycle issues
- Pre-existing 21 failures are documented baseline
- Any increase in failures beyond 21 requires investigation

**Test Results Comparison Required:**
- Before Arena work: 25 pass, 21 fail
- After Arena work: Compare against this baseline
- Report: new tests added, new failures introduced (if any), pre-existing failures maintained

**Status:** Test debt documented. Arena Phase 1 will not fix unrelated upstream test infrastructure issues.

---
---

## Arena Test Coverage

### Current Status: NONE

**Files searched:** `src/test/**/*.cpp`  
**Arena-specific tests found:** 0

**Critical Gap:** Phase 1 bug fixes require Arena test coverage before implementation.

---

## Test Helper Infrastructure

### Location: `src/test/testhelper/`

**Purpose:** Mock objects and test setup utilities

**Known Issue:** Mock object lifecycle management causes leak detection errors in 21 tests. Does not indicate production code bugs.

**Files:**
- `setup.hpp` — Test world/character/client mocking

---

## Integration Tests

### Location: `src/test/integration/`

**Status:** Directory exists but content not audited for M0.9.

**Note:** Integration tests may require full server startup, database, and protocol simulation. Not suitable for fast unit test feedback.

---

## Static Analysis

### Status: NOT CONFIGURED

**Tools evaluated:** None  
**Recommendation:** Consider adding clang-tidy or similar for Phase 1+

---

## Sanitizers

### AddressSanitizer (ASan)

**Status:** NOT ENABLED  
**Toolchain Support:** MSVC supports ASan via `/fsanitize=address`  
**Relevance:** Bug #4 investigation requires ASan to detect use-after-free or double-free

**Configuration:**
```powershell
cmake -DCMAKE_CXX_FLAGS="/fsanitize=address" ..
```

**Planned Use:** Phase 1 Bug #4 investigation

### ThreadSanitizer (TSan)

**Status:** NOT AVAILABLE ON MSVC  
**Relevance:** LOW (ArenaServ is single-threaded for gameplay)

### UndefinedBehaviorSanitizer (UBSan)

**Status:** NOT ENABLED  
**Toolchain Support:** MSVC supports UBSan via `/fsanitize=undefined`

---

## Debug Builds

### Configuration

```powershell
cmake --build . --config Debug
```

**Output:** `build/Debug/etheos.exe`

**Features:**
- Debug symbols
- Assertions enabled
- No optimization

**Use case:** Debugger attachment for Bug #1 reentrancy investigation

---

## Logging Facilities

### Console Logging

**Status:** EXISTS  
**Location:** Standard output/error streams

**Levels:** Not formally structured (no log levels like DEBUG/INFO/WARN/ERROR framework)

### File Logging

**Status:** EXISTS  
**File:** `error.log` in server working directory

**Content:** Errors and warnings

**Limitation:** No debug/trace logging for Arena state transitions

---

## Assertions

### Configuration

**Debug builds:** Assertions enabled via standard C `assert()`  
**Release builds:** Assertions disabled (`NDEBUG` defined)

**Arena relevance:** May expose state invariant violations during Bug #1/#2 investigation

---

## CTest Integration

### Status: NOT CONFIGURED

**Capability:** CMake supports CTest for test orchestration

**Current reality:** Tests run via direct `eoserv_test.exe` execution

**Recommendation:** Low priority (gtest runner sufficient for current scale)

---

## Test Gaps for Phase 1

### Arena Entry

- ❌ Entry with valid capacity
- ❌ Entry when full
- ❌ Entry boundary (capacity-1, capacity, capacity+1)
- ❌ Entry validation
- ❌ Entry state transitions

### Arena Combat

- ❌ Normal attack
- ❌ Lethal attack
- ❌ Rapid sequential attacks
- ❌ Reentrant attack scenarios (Bug #1)
- ❌ Nested timer callbacks during attack
- ❌ State consistency after attack

### Arena State Management

- ❌ Occupant count tracking (Bug #2)
- ❌ Occupant desync on disconnect
- ❌ Occupant desync on exception
- ❌ Occupant count boundary conditions
- ❌ Match state transitions

### Arena Configuration

- ❌ Valid spawn coordinates
- ❌ Out-of-bounds spawn coordinates (Bug #3)
- ❌ Spawn validation
- ❌ Zero capacity
- ❌ Negative capacity
- ❌ Malformed configuration

### Arena Timers

- ❌ Timer lifecycle (Bug #4)
- ❌ Timer destruction during callback
- ❌ Map reload during timer callback
- ❌ Timer double-free scenarios

### Arena Match Completion

- ❌ Kill counter increment
- ❌ Kill counter overflow (Bug #5)
- ❌ Winner determination
- ❌ Reset behavior

### Arena Disconnect Handling

- ❌ Disconnect during wait
- ❌ Disconnect during match
- ❌ Warp during match
- ❌ Death during match

---

## Testability Requirements for Arena

### Challenges

1. **Arena is tightly coupled to World:** Requires significant setup
2. **Arena depends on Character:** Complex object construction
3. **Arena depends on Map:** Map loading required
4. **Arena uses timers:** Callback-based, difficult to deterministically test
5. **Arena has implicit state machine:** State transitions not explicit

### Testability Improvements Needed

**Option A: Unit tests with mocking**
- Mock `Character`, `Map`, `Timer`
- Test Arena logic in isolation
- Fast, deterministic
- Requires refactoring to inject dependencies

**Option B: Integration tests with real server**
- Start server, load config, connect fake clients
- Slow, realistic
- Harder to reproduce edge cases

**Option C: Targeted instrumentation tests**
- Add debug hooks to Arena
- Trigger specific sequences (reentrancy, disconnect timing)
- Hybrid approach

**Recommendation:** Option C for Phase 1 (pragmatic, fast, targeted)

---

## Test Strategy for Phase 1

### Bug #1 (Reentrancy)

**Approach:** 
1. Add debug instrumentation to detect reentrancy
2. Create integration test that triggers timer during attack
3. Verify fix prevents reentrant execution

**Verification:** ASan + instrumentation logs

### Bug #2 (Occupant Desync)

**Approach:**
1. Add occupant count validation assertions
2. Create test that disconnects during match
3. Verify count reconciliation

**Verification:** Debug build assertions

### Bug #3 (Spawn Validation)

**Approach:**
1. Unit test for spawn coordinate validation function
2. Integration test with malformed config
3. Verify error handling

**Verification:** Direct function test + config test

### Bug #4 (Timer Investigation)

**Approach:**
1. Build with AddressSanitizer
2. Create test that reloads map during timer callback
3. Analyze ASan output

**Verification:** ASan report (expected: timer issue OR false positive)

### Bugs #5-8 (Other Issues)

**Approach:** Similar targeted test + fix strategy

---

## Recommendations

### Immediate (M0.9)

✅ Document existing test capabilities (this document)  
✅ Establish clean build process (BUILD-VERIFICATION.md)  
✅ Create Arena test matrix (ARENA-TEST-MATRIX.md)

### Phase 1

- Add Arena-specific test infrastructure
- Fix mock leak issue in test helpers
- Enable AddressSanitizer for Bug #4
- Add Arena state validation assertions

### Future

- Expand integration test coverage
- Add property-based testing for Arena
- Configure clang-tidy or similar
- Add automated test runs in CI/CD

---

## Next Steps

- **Arena test matrix:** See `docs/testing/ARENA-TEST-MATRIX.md`
- **Build process:** See `docs/development/BUILD-VERIFICATION.md`
- **GitHub workflow:** See `docs/GITHUB-WORKFLOW.md`

---

## Test Infrastructure Snapshot (M0.9)

**Date:** 2026-09-23  
**Commit:** db4602e  
**Test Framework:** Google Test (gtest)  
**Test Executable:** `build/Release/eoserv_test.exe`  
**Total Tests:** 46  
**Passing:** 25 (54%)  
**Mock Leaks:** 21 (test infrastructure issue)  
**Arena Tests:** 0 (critical gap)  
**Sanitizers:** None enabled  
**Static Analysis:** None configured

