# Phase 1 GitHub Issues

**Status:** Ready for creation  
**Date:** 2026-09-23

These issues must be created manually in GitHub (gh CLI not available).

---

## ISSUE-001: [TEST] Establish Arena Test Instrumentation

**Title:** `[TEST] Establish Arena Test Instrumentation`

**Labels:** `type: infrastructure, priority: critical, phase: 1-foundation`

**Body:**

### Objective
Create minimum test infrastructure necessary to exercise the existing legacy Arena system deterministically.

### Purpose
Enable statements like:
- "This legacy behavior existed before the fix"
- "This test failed before the fix and passes after the fix"

We need a regression harness before changing Arena behavior.

### Scope
- Arena test fixture compatible with Google Test
- Baseline behavior tests (not bug fixes)
- Minimal behavior-neutral instrumentation (if needed)
- Documentation of test architecture

### Requirements
- Investigate Arena dependency graph (Map, World, Character, Timer, config)
- Create Arena test fixture using existing Etheos test helpers where practical
- Implement minimum baseline tests
- Do NOT redesign Arena production code
- Do NOT fix bugs during test infrastructure work

### Minimum Baseline Tests
Target equivalents of:
- ARENA-CONFIG-001 (valid config initializes)
- ARENA-ENTRY-001 (first player entry)
- ARENA-ENTRY-002 (multiple players)
- ARENA-STATE-001 (waiting state)
- ARENA-STATE-002 (match transition)
- ARENA-MATCH-001 (basic match lifecycle)

### Optional: Bug Reproduction Tests
If harness permits without production changes, add reproduction tests for:
- Bug #3 (spawn validation) - expected failure
- Bug #2 (occupant desync) - expected failure
- Bug #1 (reentrancy) - expected failure
- Bug #5 (kill counter) - expected failure

Mark these as **EXPECTED FAILURE — CONFIRMED LEGACY BUG**.

### Acceptance Criteria
- [ ] Test fixture works with Google Test
- [ ] Baseline Arena tests execute
- [ ] Known bugs are NOT accidentally fixed
- [ ] Existing test debt has NOT worsened (25 pass, 21 fail baseline maintained)
- [ ] Server builds cleanly
- [ ] Smoke test passes
- [ ] Test matrix updated with implementation status
- [ ] Arena gameplay behavior NOT intentionally changed

### Dependencies
- **Depends on:** None (foundation)
- **Blocks:** #2, #3, #4, #5, #6

### Branch
`test/arena-instrumentation`

### Related Documentation
- Test matrix: docs/testing/ARENA-TEST-MATRIX.md
- Test infrastructure: docs/development/TEST-INFRASTRUCTURE.md
- Legacy bugs: docs/arena/LEGACY-ARENA-BUGS.md

---

## ISSUE-002: [BUG] Validate Arena Spawn Coordinates

**Title:** `[BUG] Validate Arena Spawn Coordinates`

**Labels:** `type: bug, priority: critical, phase: 1-foundation, component: arena`

**Body:**

### Bug Description
No validation on arena spawn coordinates from config. Out-of-bounds coordinates cause buffer overflow and crash.

### Evidence
- **Source:** `src/map.cpp:406-435`
- **Documentation:** docs/arena/LEGACY-ARENA-BUGS.md Bug #3
- **Test ID:** ARENA-CONFIG-002
- **Status:** CONFIRMED (Phase 0.5)

### Impact
- **Severity:** Critical
- **Risk:** Crash, potential buffer overflow

### Steps to Reproduce
1. Create arena config with spawn coordinates outside map bounds
2. Load map with arena
3. Observe crash or undefined behavior

### Expected Behavior
- Spawn coordinates validated against map dimensions
- Invalid config rejected with error message
- No crash on malformed config

### Actual Behavior
- No validation performed
- Out-of-bounds access occurs
- Server may crash

### Acceptance Criteria
- [ ] Validate spawn coordinates against map dimensions during config load
- [ ] Reject out-of-bounds spawns with clear error message
- [ ] Test with invalid coordinates (no crash)
- [ ] Test with valid coordinates (entry succeeds)
- [ ] Documentation updated

### Dependencies
- **Depends on:** #1 (test infrastructure)
- **Blocks:** None

### Branch
`fix/arena-spawn-validation`

### Related Documentation
- Bug analysis: docs/arena/LEGACY-ARENA-BUGS.md
- Test: ARENA-CONFIG-002

---

## ISSUE-003: [BUG] Fix Arena Occupant State Desynchronization

**Title:** `[BUG] Fix Arena Occupant State Desynchronization`

**Labels:** `type: bug, priority: critical, phase: 1-foundation, component: arena`

**Body:**

### Bug Description
Occupant counter decremented in multiple code paths without centralized management. Exception between decrement and state update causes desync, preventing new entries.

### Evidence
- **Source:** `src/character.cpp:1296-1300`, `src/character.cpp:2229-2231`
- **Documentation:** docs/arena/LEGACY-ARENA-BUGS.md Bug #2
- **Test IDs:** ARENA-STATE-001 through ARENA-STATE-004
- **Status:** CONFIRMED (Phase 0.5)

### Impact
- **Severity:** Critical
- **Risk:** State corruption, arena becomes unusable

### Steps to Reproduce
1. Player enters arena
2. Player disconnects abnormally OR exception occurs
3. Occupant count not decremented
4. Arena appears full when it is not

### Expected Behavior
- Occupant count managed centrally
- Count reconciled on disconnect/exception
- Arena state remains consistent

### Actual Behavior
- Multiple decrement paths without coordination
- Exception between decrement and state update leaves stale count
- New players cannot enter

### Acceptance Criteria
- [ ] Centralize occupant count management
- [ ] Add validation/reconciliation logic
- [ ] Test disconnect during match (count correct)
- [ ] Test normal exit (count correct)
- [ ] Test exception scenarios
- [ ] Documentation updated

### Dependencies
- **Depends on:** #1 (test infrastructure)
- **Blocks:** None

### Branch
`fix/arena-occupant-state`

### Related Documentation
- Bug analysis: docs/arena/LEGACY-ARENA-BUGS.md Bug #2
- Tests: ARENA-STATE-001 through ARENA-STATE-004

---

## ISSUE-004: [BUG] Fix Arena Attack State/Reentrancy Handling

**Title:** `[BUG] Fix Arena Attack State/Reentrancy Handling`

**Labels:** `type: bug, priority: critical, phase: 1-foundation, component: arena`

**Body:**

### Bug Description
Deferred action queue in `Arena::Attack()` operates on stale state when reentrant callbacks execute between victim identification and warp execution. Causes iterator invalidation and crashes.

**Note:** This is a reentrancy issue in single-threaded code, NOT a race condition.

### Evidence
- **Source:** `src/arena.cpp:120-210`
- **Documentation:** docs/arena/LEGACY-ARENA-BUGS.md Bug #1
- **Test IDs:** ARENA-COMBAT-004, ARENA-COMBAT-005
- **Status:** CONFIRMED (Phase 0.5)

### Impact
- **Severity:** Critical
- **Risk:** Crash, iterator invalidation, state corruption

### Steps to Reproduce
1. Player in arena match
2. Attack triggers timer callback
3. Timer callback triggers state change affecting deferred action queue
4. Original attack execution uses invalidated iterator
5. Crash or undefined behavior

### Expected Behavior
- Reentrancy prevented OR state validated before deferred actions
- No iterator invalidation
- Stable state across callbacks

### Actual Behavior
- No reentrancy guard
- Deferred actions execute on stale state
- Iterator invalidation causes crash

### Acceptance Criteria
- [ ] Add reentrancy guard OR validate actions before execution
- [ ] Test with timer callback during attack (no crash)
- [ ] Test with nested attack calls (no crash)
- [ ] State remains valid after reentrant scenario
- [ ] Documentation updated

### Dependencies
- **Depends on:** #1 (test infrastructure)
- **Blocks:** None

### Branch
`fix/arena-state-transition`

### Related Documentation
- Bug analysis: docs/arena/LEGACY-ARENA-BUGS.md Bug #1
- Execution model: docs/architecture/EXECUTION-MODEL.md
- Tests: ARENA-COMBAT-004, ARENA-COMBAT-005

---

## ISSUE-005: [RESEARCH] Investigate Arena Timer Lifetime

**Title:** `[RESEARCH] Investigate Arena Timer Lifetime`

**Labels:** `type: research, priority: high, phase: 1-foundation, component: arena`

**Body:**

### Research Question
Does arena timer destruction cause use-after-free or double-free during map reload?

### Background
Original claim: Arena accessed after destruction (use-after-free).

Phase 0.5 analysis suggests potential double-free of timer, NOT UAF of arena.

**Status:** UNPROVEN — requires runtime testing with AddressSanitizer.

### Evidence/Hypothesis
- **Source:** `src/arena.cpp:212-215`, `src/timer.cpp:314-318`
- **Documentation:** docs/arena/LEGACY-ARENA-BUGS.md Bug #4
- **Current Understanding:** Timer destruction during callback may cause issues
- **Hypothesis:** Timer double-free OR false positive

### Investigation Plan
1. Build with AddressSanitizer (`/fsanitize=address`)
2. Create test that triggers map reload during `arena_spawn()` callback
3. Create test that destroys arena with active timer
4. Analyze ASan output
5. Determine: bug exists / false positive / different issue
6. Implement fix if bug confirmed

### Expected Outcomes
- [ ] ASan build configured and functional
- [ ] Map reload test executed
- [ ] ASan report analyzed
- [ ] Findings documented in bug doc
- [ ] Bug status updated (PROVEN/DISPROVEN/DIFFERENT ISSUE)
- [ ] Fix implemented if bug confirmed
- [ ] Test passes with fix

### Dependencies
- **Depends on:** #1 (test infrastructure)
- **Blocks:** None

### Branch
`research/arena-timer-lifetime`

### Related Documentation
- Bug analysis: docs/arena/LEGACY-ARENA-BUGS.md Bug #4
- Test: ARENA-TIMER-003, ARENA-TIMER-004

---

## ISSUE-006: [RESEARCH] Assess Arena Kill Counter Overflow

**Title:** `[RESEARCH] Assess Arena Kill Counter Overflow`

**Labels:** `type: research, priority: medium, phase: 1-foundation, component: arena`

**Body:**

### Research Question
Can arena kill counter overflow with realistic usage, and if so, how should it be handled?

### Background
Kill counter may overflow with large values. Need to determine if this is a realistic issue.

### Evidence/Hypothesis
- **Source:** `src/arena.cpp` (counter type to be determined)
- **Documentation:** docs/arena/LEGACY-ARENA-BUGS.md Bug #5
- **Current Understanding:** Unknown counter data type and overflow risk
- **Hypothesis:** Overflow unlikely in realistic usage OR needs mitigation

### Investigation Plan
1. Determine counter data type (int32? int64? unsigned?)
2. Calculate realistic max kills per session
3. Assess overflow risk (time to overflow)
4. Determine appropriate fix if needed:
   - int32 → int64 (simple)
   - Clamping (prevent overflow)
   - Wrap (modulo behavior)
   - No action (unrealistic scenario)

### Expected Outcomes
- [ ] Counter type documented
- [ ] Overflow risk assessed with calculations
- [ ] Decision: FIX NEEDED or NO ACTION
- [ ] Fix implemented if warranted (upgrade type OR clamp OR wrap)
- [ ] Test with boundary values
- [ ] Documentation updated

### Dependencies
- **Depends on:** #1 (test infrastructure)
- **Blocks:** None

### Branch
`research/arena-kill-counter`

### Related Documentation
- Bug analysis: docs/arena/LEGACY-ARENA-BUGS.md Bug #5
- Tests: ARENA-COMBAT-006, ARENA-COMBAT-007

---

## Issue Creation Checklist

After manually creating these issues in GitHub:

- [ ] ISSUE-001 created
- [ ] ISSUE-002 created
- [ ] ISSUE-003 created
- [ ] ISSUE-004 created
- [ ] ISSUE-005 created
- [ ] ISSUE-006 created
- [ ] Issue numbers recorded
- [ ] Dependencies linked between issues

**Note:** These issues represent the authorized Phase 1 work. Do not create speculative future features yet.

