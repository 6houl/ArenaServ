# AI Development Rules for ArenaServ

**Status:** AUTHORITATIVE  
**Last Updated:** 2026-09-22

## Core Principles

### 1. Evidence-Based Development

**RULE:** Every technical decision must be grounded in verifiable evidence.

**DO:**
- Cite source files, functions, and line numbers
- Reference protocol specifications
- Cross-check against multiple sources
- Test against vanilla client when possible
- Document confidence levels

**DON'T:**
- Assume behavior without verification
- Guess at client capabilities
- Invent features the client doesn't support
- Present inference as fact
- Make changes without understanding impact

### 2. Client Constraint Awareness

**RULE:** The vanilla Endless Online client cannot be modified.

**DO:**
- Work within existing packet types
- Use existing UI windows creatively
- Leverage data files (EIF/ENF/ESF/ECF/EMF)
- Explore protocol-supported features
- Test against actual client behavior

**DON'T:**
- Propose features requiring client code changes
- Assume a feature is impossible without investigation
- Ignore existing client surfaces that could be repurposed
- Create packets the client doesn't understand
- Break protocol compatibility

### 3. Conservative Code Changes

**RULE:** Minimize risk and maintain stability.

**DO:**
- Read all relevant code before changes
- Understand data flow and state management
- Consider edge cases and reentrancy issues
- Add validation and bounds checking
- Write clear, commented code
- Match existing code style

**DON'T:**
- Refactor unrelated code "while you're there"
- Change behavior without explicit requirement
- Remove "dead" code without full understanding
- Introduce new dependencies unnecessarily
- Break existing functionality

### 4. Documentation First

**RULE:** Document before implementing.

**DO:**
- Update docs/ before writing code
- Create ADRs for significant decisions
- Document unknowns explicitly
- Maintain evidence trails
- Update this ruleset as needed

**DON'T:**
- Let documentation lag behind code
- Skip documenting because "it's obvious"
- Document what you think it does vs. what it does
- Leave future AI agents to rediscover constraints

## Evidence Classification System

Use these classifications for all technical claims:

### **CONFIRMED**
- Verified in source code
- Tested against vanilla client
- Cross-referenced in multiple sources
- Reproducible behavior

### **PROBABLE**
- Strong evidence from source code
- Consistent with protocol spec
- Logical deduction from confirmed facts
- Not yet validated with client

### **EXPERIMENTAL**
- Based on controlled experiments
- Reproduces under specific conditions
- May have edge cases
- Needs more validation

### **UNKNOWN**
- Insufficient evidence
- Conflicting sources
- No clear implementation
- Requires investigation

### **CLIENT-REQUIRED**
- Definitively requires client modification
- No protocol-supported alternative exists
- Confirmed impossible server-side
- Documented as such

## Feature Classification System

### **SERVER-ONLY**
Implementation requires only server code changes.

Examples:
- Formula adjustments
- Drop rate changes
- Database schema additions
- Internal state tracking
- Backend calculations

### **DATA-ASSISTED**
Implementation requires data file modifications but no client changes.

Examples:
- New items in EIF
- New NPCs in ENF
- New spells in ESF
- Map changes in EMF
- Quest scripts

### **PROTOCOL-SUPPORTED**
Uses existing packets/windows in new ways.

Examples:
- Repurposing board system for leaderboards
- Using quest dialogs for custom menus
- Shop system for reward vendors
- Message boxes for status displays

### **CLIENT-REQUIRED**
Impossible without client modification.

Examples:
- New UI panels
- New packet types client doesn't handle
- New rendering behavior
- Custom input handling

### **UNKNOWN**
Not yet classified - requires investigation.

## Development Workflow

### Phase 0: Research
1. Read all relevant documentation in docs/
2. Search codebase for related functionality
3. Cross-reference protocol specifications
4. Examine data file formats
5. Check client reference materials
6. Document findings and unknowns

### Phase 1: Design
1. Document the problem clearly
2. Research client constraints
3. Propose solution with evidence
4. Create ADR if significant
5. Get confirmation before implementation

### Phase 2: Implementation
1. Write tests first (when possible)
2. Implement with clear comments
3. Add bounds checking and validation
4. Handle error cases gracefully
5. Match existing code style

### Phase 3: Validation
1. Run unit tests
2. Test with vanilla client
3. Check edge cases
4. Verify state management
5. Test failure modes

### Phase 4: Documentation
1. Update relevant docs/
2. Add inline code comments
3. Document new packets/data formats
4. Update CLIENT-CAPABILITY-MATRIX if relevant
5. Note any discovered unknowns

## Common Pitfalls to Avoid

### ❌ "This client can't do X"
Without checking:
- All available packet types
- All window surfaces
- Data file capabilities
- Creative repurposing options

### ❌ "This is obvious from the code"
- Reentrancy issues aren't obvious
- Edge cases aren't obvious
- Client behavior isn't obvious
- Protocol constraints aren't obvious

### ❌ "Let me fix this while I'm here"
- Scope creep introduces risk
- Unrelated changes complicate review
- May introduce regressions
- Mixing concerns

### ❌ "The old code is wrong"
It might be:
- Workaround for client behavior
- Handling protocol edge case
- Legacy but still used
- Correct for reasons not immediately apparent

### ❌ "I tested it and it works"
Define "works":
- On which map?
- With how many players?
- What about disconnect during action?
- What about simultaneous actions?
- What about map reload?
- What about edge-of-map?

## Issue Creation Requirements

Every GitHub issue must contain:

1. **Evidence** - What code/protocol/behavior led to this?
2. **Scope** - What systems are affected?
3. **Client Constraint** - What can/can't the client do?
4. **Proposal** - Specific technical approach
5. **Non-Goals** - What this explicitly doesn't change
6. **Validation** - How to verify it works
7. **Documentation** - What docs need updates

## Code Review Checklist

- [ ] Cites relevant documentation
- [ ] Understands client constraints
- [ ] Handles edge cases
- [ ] Includes validation
- [ ] Matches code style
- [ ] Updates documentation
- [ ] Tests against vanilla client
- [ ] No scope creep
- [ ] Clear comments
- [ ] Error handling

## When in Doubt

1. Document the uncertainty in [research/UNKNOWN-BEHAVIOR.md](research/UNKNOWN-BEHAVIOR.md)
2. Propose controlled experiment
3. Ask for confirmation before implementation
4. Choose conservative approach
5. Add extra validation

## References

- [CLIENT-CAPABILITY-MATRIX.md](client/CLIENT-CAPABILITY-MATRIX.md)
- [PACKET-CATALOG.md](protocol/PACKET-CATALOG.md)
- [UNKNOWN-BEHAVIOR.md](research/UNKNOWN-BEHAVIOR.md)
- [ROADMAP.md](ROADMAP.md)

---

**Remember:** Future AI agents will rely on this documentation. Be thorough, honest about unknowns, and prioritize correctness over speed.

