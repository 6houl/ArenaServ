# Arena Test Matrix

**Last Updated:** 2026-09-23  
**Status:** NOT RUN (test IDs defined, implementation pending)  
**Purpose:** Comprehensive Arena behavior verification catalog

---

## Test ID Format

`ARENA-<CATEGORY>-<NUMBER>`

**Categories:**
- `CONFIG` — Configuration loading and validation
- `ENTRY` — Arena entry and capacity
- `STATE` — State management and transitions
- `COMBAT` — Attack and combat behavior
- `TIMER` — Timer lifecycle and callbacks
- `DISCONNECT` — Disconnect and error handling
- `MATCH` — Match flow and completion

---

## Configuration Tests

### ARENA-CONFIG-001: Load Valid Arena Configuration

**Purpose:** Verify arena config parsing  
**Preconditions:** Valid `arenas.ini` present  
**Actions:** Load config  
**Expected:** Arena list populated correctly  
**Verification:** Unit test / manual inspection  
**Automatable:** Yes  
**Current Result:** NOT RUN  
**Associated Issue:** Foundation work

### ARENA-CONFIG-002: Reject Out-of-Bounds Spawn Coordinates

**Purpose:** Verify Bug #3 fix  
**Preconditions:** Malformed `arenas.ini` with spawn outside map bounds  
**Actions:** Load config  
**Expected:** Error logged, config rejected  
**Verification:** Integration test  
**Automatable:** Yes  
**Current Result:** NOT RUN  
**Associated Issue:** Bug #3

### ARENA-CONFIG-003: Handle Zero Capacity

**Purpose:** Boundary validation  
**Preconditions:** Arena with capacity=0  
**Actions:** Load config  
**Expected:** Error OR capacity=1 (implementation decision)  
**Verification:** Integration test  
**Automatable:** Yes  
**Current Result:** NOT RUN

### ARENA-CONFIG-004: Handle Negative Capacity

**Purpose:** Validation  
**Preconditions:** Arena with capacity=-1  
**Actions:** Load config  
**Expected:** Error logged, config rejected  
**Verification:** Integration test  
**Automatable:** Yes  
**Current Result:** NOT RUN

---

## Entry Tests

### ARENA-ENTRY-001: First Player Entry

**Purpose:** Baseline behavior  
**Preconditions:** Empty arena, capacity > 0  
**Actions:** Player enters  
**Expected:** Entry succeeds, occupant count = 1  
**Verification:** Debug log + state inspection  
**Automatable:** Partial (requires client simulation)  
**Current Result:** NOT RUN

### ARENA-ENTRY-002: Entry Below Capacity

**Purpose:** Normal operation  
**Preconditions:** Arena with occupants < capacity  
**Actions:** Player enters  
**Expected:** Entry succeeds, occupant count incremented  
**Verification:** State inspection  
**Automatable:** Yes  
**Current Result:** NOT RUN

### ARENA-ENTRY-003: Entry At Capacity

**Purpose:** Boundary test  
**Preconditions:** Arena occupants == capacity  
**Actions:** Player attempts entry  
**Expected:** Entry rejected OR queued (design decision)  
**Verification:** Client feedback + state inspection  
**Automatable:** Partial  
**Current Result:** NOT RUN  
**Associated Issue:** Bug #8 (capacity notification UX)

### ARENA-ENTRY-004: Entry Validation

**Purpose:** Security  
**Preconditions:** Player attempts entry with invalid state  
**Actions:** Entry attempt  
**Expected:** Validation prevents entry  
**Verification:** State inspection  
**Automatable:** Yes  
**Current Result:** NOT RUN

---

## State Management Tests

### ARENA-STATE-001: Occupant Count Increment

**Purpose:** Verify Bug #2 context  
**Preconditions:** Arena with known occupant count  
**Actions:** Player enters  
**Expected:** Count incremented exactly once  
**Verification:** State inspection + assertions  
**Automatable:** Yes  
**Current Result:** NOT RUN  
**Associated Issue:** Bug #2

### ARENA-STATE-002: Occupant Count Decrement on Normal Exit

**Purpose:** Verify Bug #2 context  
**Preconditions:** Player in arena  
**Actions:** Player leaves normally  
**Expected:** Count decremented exactly once  
**Verification:** State inspection  
**Automatable:** Yes  
**Current Result:** NOT RUN  
**Associated Issue:** Bug #2

### ARENA-STATE-003: Occupant Count on Disconnect

**Purpose:** Bug #2 reproduction  
**Preconditions:** Player in arena  
**Actions:** Player disconnects abnormally  
**Expected:** Count reconciled correctly  
**Verification:** State inspection  
**Automatable:** Yes (requires disconnect simulation)  
**Current Result:** NOT RUN  
**Associated Issue:** Bug #2

### ARENA-STATE-004: Occupant Count on Exception

**Purpose:** Bug #2 edge case  
**Preconditions:** Player in arena  
**Actions:** Exception thrown during state transition  
**Expected:** Count remains consistent  
**Verification:** Instrumentation + ASan  
**Automatable:** Partial (requires forced exception)  
**Current Result:** NOT RUN  
**Associated Issue:** Bug #2

---

## Combat Tests

### ARENA-COMBAT-001: Normal Attack

**Purpose:** Baseline  
**Preconditions:** Two players in arena match  
**Actions:** Player A attacks Player B (non-lethal)  
**Expected:** Damage applied, no crash  
**Verification:** State inspection  
**Automatable:** Partial  
**Current Result:** NOT RUN

### ARENA-COMBAT-002: Lethal Attack

**Purpose:** Winner determination  
**Preconditions:** Two players in arena match, one at low HP  
**Actions:** Player A kills Player B  
**Expected:** Winner declared, match resets  
**Verification:** State inspection  
**Automatable:** Partial  
**Current Result:** NOT RUN

### ARENA-COMBAT-003: Rapid Sequential Attacks

**Purpose:** Stress test  
**Preconditions:** Multiple players in arena match  
**Actions:** Rapid attack sequence (A→B, C→A, B→C)  
**Expected:** All attacks processed correctly, no state corruption  
**Verification:** State inspection + ASan  
**Automatable:** Yes (with attack simulation)  
**Current Result:** NOT RUN

### ARENA-COMBAT-004: Reentrant Attack (Bug #1 Reproduction)

**Purpose:** Bug #1 reproduction  
**Preconditions:** Player in arena match  
**Actions:** Attack triggers timer callback which triggers another attack  
**Expected (BEFORE FIX):** Crash or state corruption  
**Expected (AFTER FIX):** Reentrancy prevented OR state remains valid  
**Verification:** ASan + state inspection  
**Automatable:** Yes (requires instrumented timer)  
**Current Result:** NOT RUN  
**Associated Issue:** Bug #1

### ARENA-COMBAT-005: Attack During Timer Callback

**Purpose:** Bug #1 variant  
**Preconditions:** Arena match active, timer callback executing  
**Actions:** Player attacks during callback  
**Expected (AFTER FIX):** Attack deferred OR state valid  
**Verification:** Debug instrumentation  
**Automatable:** Partial  
**Current Result:** NOT RUN  
**Associated Issue:** Bug #1

### ARENA-COMBAT-006: Kill Counter Increment

**Purpose:** Bug #5 context  
**Preconditions:** Player in arena match  
**Actions:** Player kills opponent  
**Expected:** Kill count incremented  
**Verification:** State inspection  
**Automatable:** Yes  
**Current Result:** NOT RUN  
**Associated Issue:** Bug #5

### ARENA-COMBAT-007: Kill Counter Overflow

**Purpose:** Bug #5 reproduction  
**Preconditions:** Player kill count at maximum integer value  
**Actions:** Player kills opponent  
**Expected:** Overflow handled gracefully (wrap, clamp, or error)  
**Verification:** Instrumented test  
**Automatable:** Yes  
**Current Result:** NOT RUN  
**Associated Issue:** Bug #5

---

## Timer Tests

### ARENA-TIMER-001: Timer Creation

**Purpose:** Baseline  
**Preconditions:** Arena configured  
**Actions:** Arena match starts, timer created  
**Expected:** Timer registered correctly  
**Verification:** Debug log  
**Automatable:** Yes  
**Current Result:** NOT RUN

### ARENA-TIMER-002: Timer Callback Execution

**Purpose:** Normal flow  
**Preconditions:** Arena timer active  
**Actions:** Timer expires  
**Expected:** Callback executes, arena state transitions  
**Verification:** State inspection  
**Automatable:** Yes  
**Current Result:** NOT RUN

### ARENA-TIMER-003: Map Reload During Timer Callback (Bug #4 Investigation)

**Purpose:** Bug #4 reproduction attempt  
**Preconditions:** Arena timer active  
**Actions:** Trigger map reload during callback execution  
**Expected (IF BUG EXISTS):** ASan detects use-after-free or double-free  
**Expected (IF NO BUG):** No ASan errors  
**Verification:** ASan report  
**Automatable:** Partial (requires forced map reload)  
**Current Result:** NOT RUN  
**Associated Issue:** Bug #4

### ARENA-TIMER-004: Arena Destruction During Timer

**Purpose:** Bug #4 variant  
**Preconditions:** Arena timer active  
**Actions:** Destroy arena before timer expires  
**Expected:** Timer cancelled cleanly  
**Verification:** ASan report  
**Automatable:** Yes  
**Current Result:** NOT RUN  
**Associated Issue:** Bug #4

---

## Disconnect Handling Tests

### ARENA-DISCONNECT-001: Disconnect While Waiting

**Purpose:** State consistency  
**Preconditions:** Player in arena waiting room  
**Actions:** Player disconnects  
**Expected:** Removed from wait list, occupant count correct  
**Verification:** State inspection  
**Automatable:** Yes  
**Current Result:** NOT RUN

### ARENA-DISCONNECT-002: Disconnect During Match

**Purpose:** Match integrity  
**Preconditions:** Player in active arena match  
**Actions:** Player disconnects  
**Expected:** Match continues OR resets (design decision), count correct  
**Verification:** State inspection  
**Automatable:** Yes  
**Current Result:** NOT RUN

### ARENA-DISCONNECT-003: Warp During Match

**Purpose:** Exit handling  
**Preconditions:** Player in active arena match  
**Actions:** Player warps to another map  
**Expected:** Match state updated, occupant count correct  
**Verification:** State inspection  
**Automatable:** Yes  
**Current Result:** NOT RUN

### ARENA-DISCONNECT-004: Death During Match

**Purpose:** Edge case  
**Preconditions:** Player in arena match  
**Actions:** Player dies (outside arena combat)  
**Expected:** Arena state reconciled  
**Verification:** State inspection  
**Automatable:** Partial  
**Current Result:** NOT RUN

---

## Match Flow Tests

### ARENA-MATCH-001: Match Start

**Purpose:** Baseline  
**Preconditions:** Sufficient players in wait queue  
**Actions:** Match start triggered  
**Expected:** Players warped to arena, match state = active  
**Verification:** State inspection  
**Automatable:** Yes  
**Current Result:** NOT RUN

### ARENA-MATCH-002: Match Completion (Normal)

**Purpose:** Winner flow  
**Preconditions:** Two players in match  
**Actions:** One player wins  
**Expected:** Winner declared, rewards processed (if any), arena resets  
**Verification:** State inspection  
**Automatable:** Partial  
**Current Result:** NOT RUN  
**Associated Issue:** Bug #7 (completion design limitation)

### ARENA-MATCH-003: Match Reset

**Purpose:** Cleanup  
**Preconditions:** Arena match completed  
**Actions:** Reset triggered  
**Expected:** State cleared, ready for next match  
**Verification:** State inspection  
**Automatable:** Yes  
**Current Result:** NOT RUN

### ARENA-MATCH-004: Match Timeout

**Purpose:** Stalemate handling  
**Preconditions:** Arena match active, no winner after time limit  
**Actions:** Timeout expires  
**Expected:** Match ends, result determined (tie, draw, etc.)  
**Verification:** State inspection  
**Automatable:** Yes  
**Current Result:** NOT RUN

---

## Edge Case Tests

### ARENA-EDGE-001: Empty Arena Server Startup

**Purpose:** Initialization  
**Preconditions:** Server starts with arena config  
**Actions:** Server starts  
**Expected:** Arena initialized correctly, no crashes  
**Verification:** Server log  
**Automatable:** Yes  
**Current Result:** NOT RUN

### ARENA-EDGE-002: Repeated Match Sequence

**Purpose:** Resource leak detection  
**Preconditions:** Arena operational  
**Actions:** Run 100 consecutive matches  
**Expected:** No memory leaks, state remains valid  
**Verification:** Memory profiling  
**Automatable:** Yes  
**Current Result:** NOT RUN

### ARENA-EDGE-003: Simultaneous Entry Attempts

**Purpose:** Concurrency (within single thread)  
**Preconditions:** Arena near capacity  
**Actions:** Multiple players attempt entry in same tick  
**Expected:** Entry serialized correctly, no count corruption  
**Verification:** State inspection + assertions  
**Automatable:** Yes  
**Current Result:** NOT RUN

---

## Test Execution Plan for Phase 1

### Priority 1: Bug Reproduction Tests

**Must run BEFORE fixes:**
- `ARENA-COMBAT-004` (Bug #1 reentrancy)
- `ARENA-STATE-003` (Bug #2 desync)
- `ARENA-CONFIG-002` (Bug #3 bounds)
- `ARENA-TIMER-003` (Bug #4 investigation)
- `ARENA-COMBAT-007` (Bug #5 overflow)

### Priority 2: Bug Verification Tests

**Must run AFTER fixes:**
- Same tests as Priority 1, expected results change to "no crash, state valid"

### Priority 3: Regression Tests

**Run before each commit:**
- All Priority 1 + Priority 2 tests
- `ARENA-ENTRY-001` (baseline sanity)
- `ARENA-COMBAT-001` (baseline sanity)

### Priority 4: Coverage Expansion

**After Phase 1:**
- All remaining test IDs
- Additional edge cases discovered during implementation

---

## Test Implementation Strategy

### Approach A: Unit Tests (Mocking)

**Pros:** Fast, deterministic, isolated  
**Cons:** Requires refactoring Arena for dependency injection  
**Suitable for:** Config validation, state management logic

### Approach B: Integration Tests (Full Server)

**Pros:** Realistic, tests actual behavior  
**Cons:** Slow, complex setup, harder to reproduce edge cases  
**Suitable for:** Match flow, disconnect handling

### Approach C: Instrumented Tests (Debug Hooks)

**Pros:** Pragmatic, targeted, fast  
**Cons:** Requires temporary instrumentation code  
**Suitable for:** Bug reproduction (reentrancy, timer lifecycle)

**Recommendation:** **Approach C for Phase 1**, transition to A+B for long-term coverage.

---

## Verification Methods

| Method | Use Case | Automation |
|--------|----------|------------|
| State Inspection | Occupant count, match state | Yes (debug hooks) |
| Debug Log | Event sequence | Yes (log parsing) |
| ASan | Memory safety (Bug #4) | Yes (ASan report) |
| Assertions | Invariant violations | Yes (debug build) |
| Client Simulation | Protocol correctness | Partial |
| Manual Testing | UX, edge cases | No |

---

## Test Matrix Summary

**Total Test IDs Defined:** 39  
**Config Tests:** 4  
**Entry Tests:** 4  
**State Tests:** 4  
**Combat Tests:** 7  
**Timer Tests:** 4  
**Disconnect Tests:** 4  
**Match Tests:** 4  
**Edge Case Tests:** 3  
**Priority 1 (Bug Reproduction):** 5  
**Executed:** 0 (NOT RUN)  
**Passing:** 0 (NOT RUN)

---

## Next Steps

1. Implement instrumentation hooks for Bug #1/#2 investigation
2. Enable AddressSanitizer for Bug #4 investigation
3. Create integration test harness
4. Execute Priority 1 tests (bug reproduction)
5. Implement fixes
6. Execute Priority 2 tests (verification)
7. Expand coverage (Priority 3+)

---

## Test Matrix Snapshot (M0.9)

**Date:** 2026-09-23  
**Commit:** db4602e  
**Test IDs Defined:** 39  
**Tests Executed:** 0  
**Status:** Test matrix established, implementation pending Phase 1

