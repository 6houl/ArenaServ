# Vanilla Client Experiment Register

**Purpose:** Track experiments requiring actual vanilla client testing  
**Status:** PENDING (Phase 0.5 - experiments not yet conducted)

## Experiments Required

### EXP-001: Quest Dialog Text Limits
**Status:** PENDING  
**Priority:** P0 (Critical for arena info display)  
**Question:** What are the maximum text limits for quest dialog?

**Test Method:**
1. Create quest with incrementally larger text
2. Test character limits per line
3. Test total line count
4. Test word wrapping behavior
5. Test special characters

**Expected Result:** Dialog truncates or rejects oversized content  
**Risk:** Server crash or client hang if limits exceeded

---

### EXP-002: Board Post Limits
**Status:** PENDING  
**Priority:** P1 (High - arena rankings)  
**Question:** How many board posts can client display?

**Test Method:**
1. Create board with 1, 10, 50, 100, 255 posts
2. Observe client rendering
3. Test scrolling behavior
4. Test post body size limits

**Expected Result:** Client has hard limit on posts  
**Risk:** Client crash or corruption with excessive posts

---

### EXP-003: Info Box Content Size
**Status:** PENDING  
**Priority:** P1 (High - match summaries)  
**Question:** Maximum content size for info box?

**Test Method:**
1. Send info box with incrementally larger content
2. Test line count limits
3. Test character count limits
4. Test scroll behavior

**Expected Result:** Content truncates or box limits apply  
**Risk:** Display corruption

---

### EXP-004: Packet Size Limits
**Status:** PENDING  
**Priority:** P0 (Critical - server stability)  
**Question:** What are the hard packet size limits?

**Test Method:**
1. Send packets with incrementally larger payloads
2. Test each packet family used for arena
3. Monitor client and server behavior

**Expected Result:** Client or server rejects oversized packets  
**Risk:** Disconnect, crash, or security issue

---

### EXP-005: Board Rendering Edge Cases
**Status:** PENDING  
**Priority:** P2 (Medium)  
**Question:** How does board render special characters, long subjects, etc.?

**Test Method:**
1. Unicode characters
2. Control characters
3. Very long subject lines
4. Empty subjects/bodies

**Expected Result:** Client sanitizes or truncates  
**Risk:** Display corruption

---

## Experiment Log

*No experiments conducted during Phase 0 or Phase 0.5 (documentation only).*

## Results (When Available)

Results will be added here after Phase 1 testing.

---

**Note:** All experiments require actual vanilla EO client (version 28) and test server instance.
