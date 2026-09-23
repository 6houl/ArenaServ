# Unknown Behavior & Research Questions

**Status:** Phase 0.5 Updated  
**Last Updated:** 2026-09-22  
**Purpose:** Catalog uncertainties requiring investigation

## Phase 0.5 Resolutions

### Resolved Questions

#### Execution Model (RESOLVED)
**Original Question:** Does the server use multi-threaded request processing?  
**Answer:** ❌ NO - Single-threaded main loop confirmed  
**Evidence:** `src/main.cpp:461-468`  
**Reference:** `docs/architecture/EXECUTION-MODEL.md`  
**Date Resolved:** 2026-09-22

#### Bug #1 Classification (RESOLVED)
**Original Question:** Is Bug #1 a traditional race condition?  
**Answer:** ❌ NO - It's a reentrancy issue with deferred actions  
**Evidence:** Single-threaded execution + action queue pattern  
**Reference:** `docs/arena/LEGACY-ARENA-BUGS.md` Bug #1  
**Date Resolved:** 2026-09-22

#### Arena Persistence (RESOLVED)
**Original Question:** Where is arena data stored in database?  
**Answer:** Nowhere - Legacy arena has ZERO persistence  
**Evidence:** `install.sql` (no arena tables)  
**Reference:** `docs/research/DATABASE-BASELINE.md`  
**Date Resolved:** 2026-09-22

---


During Phase 0 archaeology, several behaviors, capabilities, and limitations were documented as UNKNOWN, PROBABLE, or EXPERIMENTAL. This document catalogs all questions requiring further research before implementation.

---

## Classification System

| Status | Meaning | Action Required |
|--------|---------|-----------------|
| ⚠️ UNKNOWN | No evidence found | Research + testing required |
| 🔶 PROBABLE | Indirect evidence | Verification testing required |
| 🧪 EXPERIMENTAL | Theory untested | Proof-of-concept testing |
| ⚫ BLOCKED | Requires external resource | Acquire resource first |

---

## Category 1: Client Capabilities

### Q1.1: Virtual Message Boards (No Tile Required)

**Status:** 🧪 EXPERIMENTAL  
**Question:** Can message boards be triggered without Board tile interaction?

**Current Evidence:**
- Board system triggered via `BOARD, OPEN` packet
- Handler checks for Board1-8 tile in range: `src/handlers/Board.cpp:174-189`
- Tile check enforced before `character->board` set

**Theory:**
Could manually set `character->board` pointer and send packet, bypassing tile requirement?

```cpp
// Hypothetical:
character->board = world->boards[0];  // Set board pointer
character->ShowBoard();                // Send packet
```

**Testing Required:**
1. Set board pointer without tile
2. Send `BOARD, OPEN` packet
3. Observe client response
4. Test board interactions (read, create posts)

**Impact:** ⭐⭐⭐⭐⭐
- If possible: Leaderboards accessible from anywhere
- If not: Must use Board tiles on maps

**Evidence Location:** `src/handlers/Board.cpp:174-189`

---

### Q1.2: Quest Dialog Without NPC

**Status:** 🧪 EXPERIMENTAL  
**Question:** Can quest dialogs be triggered without NPC interaction?

**Current Evidence:**
- Quest system requires `character->npc` pointer: `src/handlers/Quest.cpp:103-147`
- `character->npc_type` must be set to `ENF::Quest`
- Handler validates NPC in range

**Theory:**
Could create "virtual NPC" or set pointer to trigger dialog without physical NPC?

**Testing Required:**
1. Set `character->npc` to dummy/off-map NPC
2. Set `character->npc_type = ENF::Quest`
3. Send `QUEST, DIALOG` packet
4. Test dialog interaction

**Impact:** ⭐⭐⭐⭐
- If possible: Arena menus without NPC spawns
- If not: Must place Arena Master NPCs on all maps

**Evidence Location:** `src/handlers/Quest.cpp:103-147`

---

### Q1.3: Custom Packet Data in Existing Families

**Status:** 🔶 PROBABLE  
**Question:** Can we send non-standard data in existing packet families that client ignores?

**Current Evidence:**
- Clients typically read expected bytes then stop
- Extra data likely ignored
- Risk: Client crash if expects specific size

**Theory:**
Append metadata to packets for server-side tracking without affecting client.

**Example:**
```cpp
// Standard packet
PacketBuilder builder(PACKET_BOARD, PACKET_OPEN, expected_size);
builder.Add...(standard data);

// Hypothetical: Append server metadata
builder.AddInt(internal_tracking_id);  // Client ignores?
```

**Testing Required:**
1. Send packet with extra trailing bytes
2. Verify client behavior (display vs crash)
3. Confirm data ignored gracefully

**Impact:** ⭐⭐
- Low priority - server can track separately
- Nice-to-have for compact encoding

---

### Q1.4: Maximum Message Board Posts

**Status:** ⚠️ UNKNOWN  
**Question:** What is the client-side limit for board posts displayed?

**Current Evidence:**
- Server config: `BoardMaxPosts = 50`
- Config: `AdminBoardLimit = 100`
- Client UI: Fixed size list box

**Unknown:**
- Does client have hard limit < server limit?
- Does client crash/truncate with 100+ posts?
- Scrolling behavior with large lists?

**Testing Required:**
1. Create board with 50, 100, 200 posts
2. Observe client display
3. Test scrolling, clicking
4. Find hard limit

**Impact:** ⭐⭐⭐
- Affects leaderboard capacity
- May need pagination if limited

**Evidence Location:** `config/boards.ini`, `src/character.cpp:1402-1459`

---

### Q1.5: Info Box Content Length Limit

**Status:** ⚠️ UNKNOWN  
**Question:** What is maximum content length for info boxes?

**Current Evidence:**
- Server: No hard limit enforced
- Word wrap algorithm: `INFOBOX_WIDTH = 197` pixels
- Packet: `PACKET_MESSAGE, PACKET_ACCEPT`

**Unknown:**
- Client buffer size for content?
- Practical limit before crash/truncate?
- Scrolling limits?

**Testing Required:**
1. Send info box with 1KB content
2. Send info box with 10KB content
3. Send info box with 100KB content
4. Find crash/truncate threshold

**Impact:** ⭐⭐⭐
- Affects match report detail level
- May need content truncation

**Evidence Location:** `src/character.cpp:2068-2189`

---

### Q1.6: Multiple Simultaneous Windows

**Status:** ⚠️ UNKNOWN  
**Question:** Can client display multiple windows (board + quest, shop + info box)?

**Current Evidence:**
- Most windows modal (block other actions)
- Some may allow simultaneous display

**Unknown:**
- Which window combinations work?
- Does client queue or reject second window?
- Can info box overlay quest dialog?

**Testing Required:**
1. Open quest dialog
2. Send info box packet
3. Observe client behavior
4. Test all window combinations

**Impact:** ⭐⭐
- Affects UI design flexibility
- May need sequential flows instead of overlays

---

## Category 2: Protocol & Packets

### Q2.1: PACKET_ARENA Unknown Fields

**Status:** ⚠️ UNKNOWN  
**Question:** What do unknown fields in arena packets represent?

**Current Evidence:**
```cpp
// ARENA/SPEC packet (elimination)
builder.AddShort(0);     // Unknown field 1
builder.AddByte(255);
builder.AddChar(0);      // Unknown field 2
```

**Evidence Location:** `src/arena.cpp:176-185`

**Theory:**
- Could be flags (spectator mode, special elimination)
- Could be reserved/unused
- Could be version/protocol field

**Testing Required:**
1. Modify unknown fields
2. Observe client behavior
3. Check client dissassembly/protocol docs

**Impact:** ⭐⭐
- May unlock hidden features
- Or may be vestigial

---

### Q2.2: Packet Rate Limiting

**Status:** 🔶 PROBABLE  
**Question:** What are exact rate limits for each packet family?

**Current Evidence:**
- Attack delay: `SpikeDamageDelay` config (500ms default)
- Walk speed: `WalkSpeed` config
- Spell cooldown: ESF cast_time field

**Unknown:**
- Per-packet rate limits?
- Global packet rate limit?
- Penalties for exceeding (disconnect, ignore)?

**Testing Required:**
1. Send rapid packet bursts
2. Measure accept/reject rates
3. Test disconnect thresholds

**Impact:** ⭐⭐⭐
- Affects UI responsiveness
- Anti-spam considerations

---

### Q2.3: Packet Priority & Ordering

**Status:** ⚠️ UNKNOWN  
**Question:** Are packets processed in strict FIFO order or with priority?

**Current Evidence:**
- Handler registration: `src/handlers/handlers.cpp`
- Queue flag: `OutOfBand` for non-queued packets
- State check: `StateCheck()` validates before handling

**Unknown:**
- Do some packets jump queue?
- Are warp packets prioritized?
- Attack vs movement ordering?

**Testing Required:**
1. Send mixed packet types rapidly
2. Observe execution order
3. Check for reordering behavior

**Impact:** ⭐⭐⭐
- Affects race condition risks
- Important for real-time arena combat

**Evidence Location:** `src/handlers/handlers.hpp:58-62`

---

## Category 3: Data Files & Content

### Q3.1: Maximum Item/NPC/Spell IDs

**Status:** ⚠️ UNKNOWN  
**Question:** What are the actual maximum IDs for each pub file type?

**Current Evidence:**
- IDs stored as `int` (server-side): up to 2^31-1
- Protocol often uses `short`: up to 32,767
- Length field: 2 bytes = up to 65,535 records

**Unknown:**
- Client hard limits?
- ID 0 reserved?
- Gaps in ID ranges allowed?

**Testing Required:**
1. Create item with ID 10,000
2. Create item with ID 60,000
3. Test client load and display
4. Find crash threshold

**Impact:** ⭐⭐⭐
- Affects ID allocation strategy
- Arena items use 5000+ range

**Evidence Location:** `src/eodata.hpp`

---

### Q3.2: Item Type Behavior Overrides

**Status:** 🧪 EXPERIMENTAL  
**Question:** Can server override item type behavior client-side?

**Current Evidence:**
- Item types defined: `EIF::Type` enum
- Client interprets types (Heal, Weapon, etc.)
- Server can validate/modify effects

**Theory:**
- Server could ignore type, implement custom behavior
- Client sees "Heal" item, server treats as "Arena Token"
- Risk: Client confusion, UI mismatch

**Testing Required:**
1. Create Static item (no use effect)
2. Server handles USE packet with custom logic
3. Test client response

**Impact:** ⭐⭐⭐⭐
- If possible: Arena tokens with custom redemption
- If not: Must use existing item behaviors

---

### Q3.3: Map Tile Spec Client Recognition

**Status:** 🔶 PROBABLE  
**Question:** Does client recognize all TileSpec values or only specific ones?

**Current Evidence:**
- TileSpec enum: 37 defined values
- Some unknown: `Unknown1-7`
- Client behavior varies by spec

**Unknown:**
- Do unknown specs crash client?
- Are they ignored (walkable)?
- Can we repurpose unknown specs?

**Testing Required:**
1. Create map with Unknown1-7 tiles
2. Test walkability
3. Test client rendering
4. Check for crashes

**Impact:** ⭐⭐
- May unlock extra tile types
- Or confirm unknowns are reserved

**Evidence Location:** `src/map.hpp:76-115`

---

### Q3.4: NPC Vendor ID Reuse

**Status:** 🔶 PROBABLE  
**Question:** Can multiple NPCs share same vendor_id for quest/shop?

**Current Evidence:**
- `vendor_id` field in ENF
- Quest system looks up by vendor_id
- Shop system looks up by vendor_id

**Theory:**
- Multiple NPCs (different maps) → same arena menu
- Reduces quest file duplication

**Testing Required:**
1. Create 2 NPCs with same vendor_id
2. Interact with both
3. Verify same quest/shop opens

**Impact:** ⭐⭐⭐
- If works: Easy to place Arena Master on all maps
- If not: Must create per-map NPCs

---

## Category 4: Game Mechanics

### Q4.1: Combat Damage Calculation Details

**Status:** 🔶 PROBABLE  
**Question:** What is the exact damage formula?

**Current Evidence:**
- Code: `src/map.cpp:1487-1650` (Attack function)
- Uses weapon stats, character stats, armor
- Random variance applied

**Unknown:**
- Exact formula coefficients
- Critical hit mechanics
- Armor mitigation formula
- Evasion calculation

**Testing Required:**
1. Controlled combat tests
2. Log inputs and outputs
3. Reverse-engineer formula
4. Validate edge cases

**Impact:** ⭐⭐⭐⭐
- Critical for arena balance
- Affects matchmaking fairness

---

### Q4.2: Spell Targeting Edge Cases

**Status:** ⚠️ UNKNOWN  
**Question:** How do group/AoE spells select targets in PvP?

**Current Evidence:**
- Spell target types: Normal, Self, Group
- Target restrictions: NPCOnly, Friendly, Opponent
- Cast packets: `PACKET_CAST, PACKET_TARGET_GROUP`

**Unknown:**
- Group spell in arena → hits allies or enemies?
- Target filter in PvP context?
- Range limits?

**Testing Required:**
1. Cast group heal in arena
2. Cast group damage in arena
3. Observe who gets hit
4. Test Friendly vs Opponent restrictions

**Impact:** ⭐⭐⭐⭐
- Affects arena ruleset design
- May need to disable certain spells

**Evidence Location:** `src/handlers/Spell.cpp`

---

### Q4.3: Party System in Arena Context

**Status:** ⚠️ UNKNOWN  
**Question:** How does party system interact with arena combat?

**Current Evidence:**
- Party packet family: `PACKET_PARTY`
- Party buffs, shared EXP
- Friendly fire settings?

**Unknown:**
- Can party members attack each other in arena?
- Do party heals work in arena?
- Team arena mode possible?

**Testing Required:**
1. Form party
2. Enter arena together
3. Test attack, heal, buffs
4. Check friendly fire

**Impact:** ⭐⭐⭐⭐⭐
- Enables team arena modes (2v2, 3v3)
- Critical for feature roadmap

**Evidence Location:** `src/handlers/Party.cpp`

---

### Q4.4: Item Durability & Arena Equipment

**Status:** 🔶 PROBABLE  
**Question:** Do items degrade during arena combat?

**Current Evidence:**
- Item durability not explicitly in EIF structure
- Unknown if implemented server-side
- Unknown if arena affects durability

**Testing Required:**
1. Enter arena with equipment
2. Fight N matches
3. Check item state
4. Compare durability before/after

**Impact:** ⭐⭐⭐
- Affects arena cost/rewards
- May need durability exemption

---

## Category 5: Performance & Limits

### Q5.1: Maximum Players Per Map

**Status:** ⚠️ UNKNOWN  
**Question:** What is the practical limit for concurrent players on one map?

**Current Evidence:**
- No hard limit in code
- Performance degrades with broadcast packets
- Client rendering limits?

**Testing Required:**
1. Spawn 10, 50, 100 players on map
2. Measure performance
3. Test client responsiveness
4. Find crash/lag threshold

**Impact:** ⭐⭐⭐⭐
- Affects FFA arena capacity
- Tournament spectator limits

---

### Q5.2: Database Query Performance

**Status:** ⚠️ UNKNOWN  
**Question:** Can database handle real-time ELO/stats queries during matchmaking?

**Current Evidence:**
- Database abstraction: SQL Server, MySQL, SQLite
- No arena-specific queries yet
- General queries for character load/save

**Unknown:**
- Query latency for 1000+ player rankings?
- Index optimization needed?
- Caching strategy required?

**Testing Required:**
1. Create large player table (10,000+ records)
2. Query rankings by ELO
3. Measure latency
4. Test concurrent queries

**Impact:** ⭐⭐⭐⭐
- Affects matchmaking responsiveness
- Leaderboard update frequency

---

### Q5.3: Timer System Precision

**Status:** 🔶 PROBABLE  
**Question:** What is the precision of the timer system for match timers?

**Current Evidence:**
- `Timer` class: `src/timer.cpp`
- Uses system time
- TimeEvent registration

**Unknown:**
- Tick rate?
- Minimum interval?
- Drift over long matches?

**Testing Required:**
1. Create 1-second timer
2. Measure actual intervals
3. Test 10-minute match timer
4. Check for drift

**Impact:** ⭐⭐⭐
- Affects match timing accuracy
- Round timer precision

**Evidence Location:** `src/timer.cpp`, `src/timer.hpp`

---

## Category 6: Client-Server Sync

### Q6.1: State Desync Detection

**Status:** ⚠️ UNKNOWN  
**Question:** How can server detect client state desync?

**Current Evidence:**
- No explicit sync validation in code
- Client trusts server state
- Server trusts client position (with validation)

**Unknown:**
- Can server request client state snapshot?
- Periodic sync checks?
- Desync recovery mechanism?

**Testing Required:**
1. Force desync (manual packet injection)
2. Observe client behavior
3. Test recovery methods
4. Implement sync check

**Impact:** ⭐⭐⭐⭐
- Critical for arena integrity
- Anti-cheat considerations

---

### Q6.2: Reconnection During Match

**Status:** ⚠️ UNKNOWN  
**Question:** What happens if player disconnects during arena match?

**Current Evidence:**
- Disconnect: `Character::~Character()` decrements occupants
- No reconnection logic
- Character destroyed on disconnect

**Unknown:**
- Can player rejoin same match?
- Is match state preserved?
- Automatic forfeit or pause?

**Testing Required:**
1. Player enters arena
2. Simulate disconnect
3. Attempt reconnect
4. Check match state

**Impact:** ⭐⭐⭐⭐⭐
- Critical for competitive play
- Affects match outcome fairness

**Evidence Location:** `src/character.cpp:2229-2231`

---

## Category 7: Edge Cases & Exploits

### Q7.1: Simultaneous Elimination Race Condition

**Status:** ⚠️ UNKNOWN (Known Bug, Unknown If Exploitable)  
**Question:** Can players intentionally trigger race condition for draw?

**Current Evidence:**
- Bug documented: `LEGACY-ARENA-BUGS.md` #1
- Happens with simultaneous attacks
- Results in double elimination

**Unknown:**
- Can players coordinate to trigger?
- Client-side input buffering?
- Server tick rate affects?

**Testing Required:**
1. Two players attack simultaneously
2. Attempt to reproduce consistently
3. Test exploit potential
4. Measure tick timing

**Impact:** ⭐⭐⭐⭐
- Exploitable = high priority fix
- If rare = lower priority

---

### Q7.2: Arena Boundary Escape

**Status:** 🧪 EXPERIMENTAL  
**Question:** Can players leave arena combat area without being warped out?

**Current Evidence:**
- Arena combat checks `character->arena != NULL`
- No forced boundaries within map
- Players can walk anywhere

**Theory:**
- Players could hide outside arena
- Wait out opponents
- Avoid combat

**Testing Required:**
1. Enter arena
2. Walk to map edges
3. Test if combat still works
4. Check if match can end

**Impact:** ⭐⭐⭐⭐
- If possible: Griefing vector
- Needs boundaries or timeout

---

### Q7.3: Item Duplication via Arena

**Status:** 🔶 PROBABLE (Not Exploitable)  
**Question:** Can arena mechanics duplicate items?

**Current Evidence:**
- Arena doesn't modify inventory
- No item rewards in legacy system
- Warp preserves inventory

**Theory:**
- Low risk - no item manipulation
- But future reward system could introduce bugs

**Testing Required:**
1. Enter arena with items
2. Get eliminated
3. Check inventory
4. Verify no duplication

**Impact:** ⭐⭐⭐⭐⭐ (If vulnerable)
- Economy-breaking if exploitable

---

## Research Priority Matrix

| Priority | Questions | Justification |
|----------|-----------|---------------|
| P0 (Critical) | Q6.2, Q4.3, Q7.1, Q7.3 | Affects core arena functionality |
| P1 (High) | Q1.1, Q1.2, Q4.1, Q5.1, Q6.1 | Major feature enablers |
| P2 (Medium) | Q1.4, Q1.5, Q3.2, Q4.2, Q5.2 | Important for polish |
| P3 (Low) | Q1.3, Q1.6, Q2.1, Q3.1, Q3.3 | Nice-to-have features |
| P4 (Deferred) | Q2.2, Q2.3, Q3.4, Q4.4, Q5.3 | Can work around |

---

## Research Methodology

### General Approach

#### 1. Code Analysis
- Grep for relevant code paths
- Read handler implementations
- Trace packet flows

#### 2. Controlled Testing
- Minimal test cases
- Isolated variables
- Reproducible scenarios

#### 3. Client Observation
- Monitor client behavior
- Log client responses
- Check for crashes

#### 4. Protocol Inspection
- Packet capture (if needed)
- Compare with protocol spec
- Validate against client

### Test Environment Setup

```bash
# Minimal test server
1. Fresh ArenaServ install
2. Empty database
3. Test client (vanilla)
4. Network monitoring (optional)
```

### Documentation Process

For each research question:
1. **Test** - Execute controlled experiments
2. **Record** - Log results with evidence
3. **Classify** - Update status (CONFIRMED, IMPOSSIBLE, etc.)
4. **Document** - Add findings to relevant docs
5. **Decide** - Update roadmap based on findings

---

## External Resources Needed

### ⚫ BLOCKED: Client Dissassembly

**Questions Blocked:** Q2.1, Q3.3, many client-side limits

**Resource:** Endless.exe disassembly or source code  
**Location:** `references/Client/Endless.exe`  
**Tool Needed:** IDA Pro, Ghidra, or similar  
**Skill Needed:** x86 assembly, reverse engineering

**Alternatives:**
- EOSource client source (reference implementation)
- Protocol documentation reverse-engineered by community
- Trial-and-error testing

---

### ⚫ BLOCKED: Official Protocol Documentation

**Questions Blocked:** Packet field meanings, limits

**Resource:** Official Endless Online protocol spec  
**Status:** Community reverse-engineered specs exist  
**Location:** `references/eo-protocol-master/`

**Note:** XML protocol specs available but may be incomplete

---

### ⚫ BLOCKED: Live Test Players

**Questions Blocked:** Q5.1, Q6.2, Q7.1 (concurrent testing)

**Resource:** Multiple test clients / players  
**Workaround:** Bot framework (EOBot), multi-client tools

---

## Research Tracking

### Template for Investigation Results

```markdown
## [Question ID]: [Title]

**Investigation Date:** YYYY-MM-DD  
**Researcher:** [Name]  
**Status:** ✅ CONFIRMED | ❌ IMPOSSIBLE | 🔶 PARTIAL

### Test Setup
- Server version: X.X.X
- Client version: X.X.X
- Test scenario: [description]

### Test Results
[Detailed findings]

### Conclusion
[Answer to question]

### Evidence
- Code: `file.cpp:line`
- Logs: [paste relevant logs]
- Screenshots: [if applicable]

### Impact on Roadmap
[How this affects planned features]
```

---

## Summary

**Total Questions:** 29  
**Critical Priority (P0):** 4  
**High Priority (P1):** 5  
**Medium Priority (P2):** 5  
**Low Priority (P3):** 6  
**Deferred (P4):** 5  
**Blocked:** 4  

**Key Unknowns:**
1. Virtual windows (boards/quests without tiles)
2. Party system interaction with arena
3. Reconnection handling
4. Performance limits (players, queries)
5. Exploit potential (race conditions, boundaries)

**Recommendation:** Prioritize P0-P1 questions in Phase 1 before major implementation work. Many questions can be answered with simple controlled tests. Blocked questions requiring reverse engineering can be deferred or worked around.

---

**Next Steps:** After Phase 0 complete, create Phase 1 research sprint to systematically test and document answers to critical unknowns.

