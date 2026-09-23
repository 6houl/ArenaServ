# Phase 1 — Proposed Issues

**Last Updated:** 2026-09-23  
**Status:** PROPOSED (not yet created in GitHub)  
**Purpose:** Define Phase 1 implementation sequence based on M0.9 testability analysis

---

## Phase 1 Objective

Establish stable Arena foundation by fixing confirmed bugs and completing critical research.

**Completion Criteria:**
- Bugs #1, #2, #3 fixed and verified
- Bug #4 investigation complete (proven/disproven)
- Bug #5 assessed
- Test infrastructure operational

---

## Issue Sequence

### Foundation Work (Must Complete First)

#### ISSUE-001: [TEST] Establish Arena Test Instrumentation

**Type:** Infrastructure  
**Priority:** Critical (blocks all bug fixes)  
**Phase:** 1-foundation

**Description:**
Add minimal debug instrumentation to Arena to support bug reproduction and verification testing without changing gameplay behavior.

**Scope:**
- Occupant count validation hooks (Bug #2)
- Reentrancy detection (Bug #1)
- Timer lifecycle logging (Bug #4)
- State transition logging

**Acceptance Criteria:**
- [ ] Instrumentation added
- [ ] Build succeeds
- [ ] No gameplay behavior changes
- [ ] Debug logs verify Arena events
- [ ] Documented in TEST-INFRASTRUCTURE.md

**Estimated Effort:** See acceptance criteria and dependencies  
**Dependencies:** None  
**Branch:** `test/arena-instrumentation`

---

### Bug Fixes (Dependency Order)

#### ISSUE-002: [BUG] Fix Arena Spawn Coordinate Validation (Bug #3)

**Type:** Bug  
**Priority:** Critical (crash risk)  
**Phase:** 1-foundation

**Description:**
No validation on arena spawn coordinates from config. Out-of-bounds coordinates cause buffer overflow and crash.

**Evidence:**
- Source: `src/map.cpp:406-435`
- Bug doc: `docs/arena/LEGACY-ARENA-BUGS.md` Bug #3
- Test: `ARENA-CONFIG-002`

**Acceptance Criteria:**
- [ ] Validate spawn coordinates against map dimensions
- [ ] Reject invalid config with error message
- [ ] Test with out-of-bounds config (no crash)
- [ ] Test with valid config (entry succeeds)
- [ ] Documentation updated

**Estimated Effort:** See acceptance criteria and dependencies  
**Dependencies:** ISSUE-001 (test infrastructure)  
**Branch:** `fix/arena-spawn-validation`

---

#### ISSUE-003: [BUG] Fix Arena Occupant Count Desync (Bug #2)

**Type:** Bug  
**Priority:** Critical (state corruption)  
**Phase:** 1-foundation

**Description:**
Occupant counter decremented in multiple code paths without centralized management. Exception between decrement and state update causes desync, preventing new entries.

**Evidence:**
- Source: `src/character.cpp:1296-1300`, `src/character.cpp:2229-2231`
- Bug doc: `docs/arena/LEGACY-ARENA-BUGS.md` Bug #2
- Test: `ARENA-STATE-001` through `ARENA-STATE-004`

**Acceptance Criteria:**
- [ ] Centralize occupant count management
- [ ] Add validation/reconciliation
- [ ] Test disconnect during match (count correct)
- [ ] Test normal exit (count correct)
- [ ] Test exception scenarios
- [ ] Documentation updated

**Estimated Effort:** See acceptance criteria and dependencies  
**Dependencies:** ISSUE-001 (test infrastructure)  
**Branch:** `fix/arena-occupant-state`

---

#### ISSUE-004: [BUG] Fix Arena Attack Reentrancy Issue (Bug #1)

**Type:** Bug  
**Priority:** Critical (crash, state corruption)  
**Phase:** 1-foundation

**Description:**
Deferred action queue in `Arena::Attack()` operates on stale state when reentrant callbacks execute between victim identification and warp execution. Causes iterator invalidation and crashes.

**Evidence:**
- Source: `src/arena.cpp:120-210`
- Bug doc: `docs/arena/LEGACY-ARENA-BUGS.md` Bug #1
- Test: `ARENA-COMBAT-004`, `ARENA-COMBAT-005`

**Note:** This is a reentrancy issue in single-threaded code, NOT a race condition.

**Acceptance Criteria:**
- [ ] Add reentrancy guard OR validate actions before execution
- [ ] Test with timer callback during attack (no crash)
- [ ] Test with nested attack calls (no crash)
- [ ] State remains valid after reentrant scenario
- [ ] Documentation updated

**Estimated Effort:** See acceptance criteria and dependencies  
**Dependencies:** ISSUE-001 (test infrastructure)  
**Branch:** `fix/arena-state-transition`

---

### Research Work

#### ISSUE-005: [RESEARCH] Investigate Bug #4 Timer Destruction

**Type:** Research  
**Priority:** High  
**Phase:** 1-foundation

**Description:**
Original claim: Arena accessed after destruction (use-after-free). Phase 0.5 analysis suggests potential double-free of timer, NOT UAF of arena. Requires runtime testing with AddressSanitizer to confirm.

**Evidence:**
- Source: `src/arena.cpp:212-215`, `src/timer.cpp:314-318`
- Bug doc: `docs/arena/LEGACY-ARENA-BUGS.md` Bug #4
- Test: `ARENA-TIMER-003`, `ARENA-TIMER-004`

**Investigation Plan:**
1. Build with AddressSanitizer (`/fsanitize=address`)
2. Create test that triggers map reload during `arena_spawn()` callback
3. Analyze ASan output
4. Determine if bug exists, is false positive, or is different issue
5. Document findings
6. Implement fix if needed

**Acceptance Criteria:**
- [ ] ASan build configured
- [ ] Map reload test executed
- [ ] ASan report analyzed
- [ ] Findings documented in bug doc
- [ ] Fix implemented if bug confirmed
- [ ] Test passes with fix

**Estimated Effort:** See acceptance criteria and dependencies  
**Dependencies:** ISSUE-001 (test infrastructure)  
**Branch:** `research/arena-timer-lifetime`

---

#### ISSUE-006: [RESEARCH] Assess Bug #5 Kill Counter Overflow

**Type:** Research  
**Priority:** Medium  
**Phase:** 1-foundation

**Description:**
Kill counter may overflow with large values. Determine if this is a realistic issue and implement appropriate handling.

**Evidence:**
- Source: `src/arena.cpp` (counter type TBD)
- Bug doc: `docs/arena/LEGACY-ARENA-BUGS.md` Bug #5
- Test: `ARENA-COMBAT-006`, `ARENA-COMBAT-007`

**Investigation Plan:**
1. Determine counter data type
2. Calculate realistic max kills per session
3. Assess overflow risk
4. Implement fix if needed (int32→int64 OR clamping OR wrap)

**Acceptance Criteria:**
- [ ] Counter type documented
- [ ] Overflow risk assessed
- [ ] Fix implemented if needed
- [ ] Test with boundary values
- [ ] Documentation updated

**Estimated Effort:** See acceptance criteria and dependencies  
**Dependencies:** None  
**Branch:** `research/arena-kill-counter`

---

## Issue NOT Created (Design Limitations)

The following are documented design limitations, NOT bugs. No Phase 1 issues:

- **Bug #6:** Single spawn point (design, not bug)
- **Bug #7:** Match only ends on death (design, not bug)
- **Bug #8:** No capacity notification (UX, not bug)

These may become Phase 2+ feature requests if user feedback warrants changes.

---

## Dependency Graph

```
ISSUE-001 (Test Infrastructure)
    ├─→ ISSUE-002 (Bug #3: Spawn validation)
    ├─→ ISSUE-003 (Bug #2: Occupant desync)
    ├─→ ISSUE-004 (Bug #1: Reentrancy)
    ├─→ ISSUE-005 (Bug #4: Timer investigation)
    └─→ ISSUE-006 (Bug #5: Kill counter)
```

**Parallel Execution:** After ISSUE-001 completes, ISSUE-002 through ISSUE-006 can proceed in parallel.

---

## Implementation Order Recommendation

## Success Metrics

### Phase 1 Complete When:

- ✅ All 6 issues resolved
- ✅ Bug #1, #2, #3 fixed and verified
- ✅ Bug #4 investigation complete (proven/disproven/fixed)
- ✅ Bug #5 assessed (fixed if warranted)
- ✅ Test infrastructure operational
- ✅ All changes merged to main
- ✅ Build succeeds
- ✅ Tests pass
- ✅ Documentation updated

---

## GitHub Issue Creation

**Status:** PROPOSED, NOT YET CREATED

**Next Step:** Await user authorization to create issues in GitHub.

**Creation Command (when authorized):**
```bash
gh issue create --title "[TEST] Establish Arena Test Instrumentation" \
                --body-file issue-001.md \
                --label "type: infrastructure,priority: critical,phase: 1-foundation"
```

---

## Phase 1 Completion Report Template

```markdown
# Phase 1 Completion Report

**Date:** [date]  
**Status:** COMPLETE  
**Issues Resolved:** 6/6

## Bugs Fixed
- ✅ Bug #1: Reentrancy (ISSUE-004)
- ✅ Bug #2: Occupant desync (ISSUE-003)
- ✅ Bug #3: Spawn validation (ISSUE-002)

## Research Complete
- ✅ Bug #4: [PROVEN/DISPROVEN/FIXED]
- ✅ Bug #5: [FIXED/NO ACTION NEEDED]

## Test Infrastructure
- ✅ Instrumentation operational
- ✅ Test matrix implemented
- ✅ ASan configured

## Verification
- ✅ All tests pass
- ✅ Build succeeds
- ✅ No regressions
- ✅ Documentation updated

## Metrics
- Commits: [count]
- Lines changed: [count]
- Test coverage: [test IDs executed]
- Bugs remaining: [count]

## Next Phase
Phase 2: Core Arena Features (matchmaking, queue, modes)
```

---

## Proposed Issues Snapshot (M0.9)

**Date:** 2026-09-23  
**Commit:** db4602e (M0.9 baseline)  
**Issues Defined:** 6  
**Issues Created:** 0 (awaiting authorization)  
**Estimated Effort:** See acceptance criteria and dependencies  
**Dependencies:** Linear (ISSUE-001 first, then parallel)

