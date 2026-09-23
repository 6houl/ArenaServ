# ArenaServ Development Roadmap

**Status:** PLANNING  
**Last Updated:** 2026-09-22  
**Purpose:** Phased development plan based on Phase 0 findings

## Overview

This roadmap outlines a structured approach to building modern arena features on ArenaServ, informed by Phase 0 archaeological research. Each phase builds on previous work with clear goals, deliverables, and success criteria.

---

## Guiding Principles

### 1. Evidence-Based Development
- ✅ **DO:** Build on CONFIRMED capabilities
- ⚠️ **VERIFY:** Test PROBABLE theories before relying on them
- ❌ **AVOID:** Assuming capabilities without evidence

### 2. Incremental Value Delivery
- Each phase delivers working, testable features
- No phase depends on incomplete future work
- Users benefit from each phase completion

### 3. Risk Mitigation First
- Fix critical bugs before adding features
- Research unknowns before design commitments
- Validate assumptions with proof-of-concepts

### 4. Vanilla Client Constraint
- All features work with unmodified client
- Creative reuse of existing systems
- Server-side implementation only

---

## Phase Summaries

| Phase | Focus | Duration | Risk |
|-------|-------|----------|------|
| **Phase $1** | $2 | TBD | Low |
| **Phase $1** | $2 | TBD | Medium |
| **Phase $1** | $2 | TBD | Medium |
| **Phase $1** | $2 | TBD | Low |
| **Phase $1** | $2 | TBD | Medium |
| **Phase $1** | $2 | TBD | High |

**Total Estimated Time:** Based on completion criteria, not calendar estimates

---

## Phase 0: Repository Archaeology ✅ COMPLETE

**Status:** ✅ COMPLETE  
**Duration:** TBD (entry/exit criteria below)  
**Goal:** Create comprehensive knowledge base

### Deliverables ✅

- [x] Project overview and development rules
- [x] Upstream lineage and build environment docs
- [x] Client capability matrix (22 capabilities)
- [x] Window system deep-dive (5 window types)
- [x] Packet catalog (51 families, 40+ actions)
- [x] Legacy arena architecture and bugs (8 critical bugs)
- [x] Data file format reference (EIF/ENF/ESF/ECF/EMF)
- [x] Unknown behavior catalog (29 research questions)
- [x] Development roadmap and GitHub workflow

### Key Findings

✅ **Confirmed Capabilities:**
- Message boards → Leaderboards
- Quest dialogs → Arena menus
- Info boxes → Match reports
- Shop system → Reward vendor
- Custom items/NPCs work with vanilla client

⚠️ **Critical Bugs Identified:**
- Reentrancy issue in attack processing
- Occupant count desynchronization
- No bounds validation on spawns
- Timer cleanup use-after-free
- Kill count overflow (char type)

🔍 **Critical Unknowns:**
- Virtual windows without tiles?
- Party system in arena?
- Reconnection during match?
- Performance limits (players, queries)?

### Success Criteria ✅

- [x] Comprehensive documentation created
- [x] All source code locations mapped
- [x] Client capabilities cataloged
- [x] Legacy bugs documented
- [x] Unknown behaviors identified

---

## Phase 1: Foundation & Bug Fixes

**Status:** 🔜 NEXT  
**Duration:** TBD (entry/exit criteria below)  
**Goal:** Fix critical bugs, research unknowns, establish infrastructure

### Prerequisites
- Phase 0 complete ✅
- Test environment setup
- Database schema designed

### Critical Bug Fixes (P0)

#### 1.1: Fix Use-After-Free in Timer Cleanup
**File:** `src/arena.cpp:212-215`  
**Priority:** 🔴 CRITICAL (Crash risk)

**Task:**
```cpp
// Unregister timer before deletion
Arena::~Arena() {
    this->map->world->timer.Unregister(this->spawn_timer);
    delete this->spawn_timer;
}
```

**Testing:** Reload map during timer callback, verify no crash

#### 1.2: Fix Bounds Validation on Spawn Coordinates
**File:** `src/map.cpp:406-435`  
**Priority:** 🔴 CRITICAL (Crash risk)

**Task:**
```cpp
// Validate spawn coordinates
if (s->sx >= this->width || s->sy >= this->height
    || s->dx >= this->width || s->dy >= this->height) {
    Console::Err("Arena spawn out of bounds");
    delete s;
    continue;
}
```

**Testing:** Config with out-of-bounds spawns, verify graceful rejection

#### 1.3: Fix Reentrancy issue in attack processing
**File:** `src/arena.cpp:120-210`  
**Priority:** 🔴 CRITICAL (Data corruption)

**Task:**
```cpp
// Immediate state update before checks
victim->arena = NULL;
--this->arena->occupants;

// NOW check match end
if (this->arena->occupants == 1) {
    // Winner declaration
}
```

**Testing:** Two players attack simultaneously, verify single winner

#### 1.4: Fix Occupant Count Desynchronization
**File:** `src/character.cpp:1296-1308`  
**Priority:** 🔴 CRITICAL (State corruption)

**Task:**
```cpp
// Transactional state updates with validation
void Arena::ReconcileOccupants() {
    int actual = 0;
    UTIL_FOREACH(this->map->characters, character) {
        if (character->arena == this)
            ++actual;
    }
    
    if (actual != this->occupants) {
        Console::Wrn("Arena occupant desync: %d != %d", 
                     this->occupants, actual);
        this->occupants = actual;
    }
}
```

**Testing:** Disconnect during match, reload map, verify count accuracy

### Research Sprint (P0-P1 Unknowns)

#### 1.5: Test Virtual Windows
**Questions:** Q1.1 (boards), Q1.2 (quests)  
**Priority:** ⭐⭐⭐⭐⭐

**Tests:**
1. Set board pointer without tile, send packet
2. Set NPC pointer without interaction, send packet
3. Document results

**Decision Point:**
- ✅ If works: Proceed with virtual window architecture
- ❌ If fails: Use tile-based approach, place NPCs on maps

#### 1.6: Test Party System in Arena
**Question:** Q4.3  
**Priority:** ⭐⭐⭐⭐⭐

**Tests:**
1. Form party, enter arena
2. Test friendly fire
3. Test party buffs/heals
4. Document behavior

**Decision Point:**
- Enables/disables team arena modes (2v2, 3v3)

#### 1.7: Test Reconnection Handling
**Question:** Q6.2  
**Priority:** ⭐⭐⭐⭐⭐

**Tests:**
1. Enter arena
2. Simulate disconnect
3. Reconnect
4. Document match state

**Decision Point:**
- Affects competitive play design
- May need match pause/resume system

#### 1.8: Test Performance Limits
**Question:** Q5.1  
**Priority:** ⭐⭐⭐⭐

**Tests:**
1. Spawn 50+ players on map
2. Measure lag/crashes
3. Test client responsiveness
4. Document limits

**Decision Point:**
- Sets FFA arena capacity
- Spectator limits

### Infrastructure Setup

#### 1.9: Database Schema Design
**Priority:** 🟡 HIGH

**Tables:**
```sql
-- Arena player stats
CREATE TABLE arena_players (
    character_id INT PRIMARY KEY,
    elo_rating INT DEFAULT 1200,
    total_matches INT DEFAULT 0,
    wins INT DEFAULT 0,
    losses INT DEFAULT 0,
    kills INT DEFAULT 0,
    deaths INT DEFAULT 0,
    current_streak INT DEFAULT 0,
    best_streak INT DEFAULT 0,
    created_at TIMESTAMP,
    updated_at TIMESTAMP
);

-- Match history
CREATE TABLE arena_matches (
    match_id INT PRIMARY KEY AUTO_INCREMENT,
    map_id INT,
    match_type VARCHAR(20), -- '1v1', 'ffa', etc.
    winner_id INT,
    start_time TIMESTAMP,
    end_time TIMESTAMP,
    duration INT
);

-- Match participants
CREATE TABLE arena_match_players (
    match_id INT,
    character_id INT,
    kills INT,
    deaths INT,
    damage_dealt INT,
    damage_taken INT,
    elo_change INT,
    placement INT,
    PRIMARY KEY (match_id, character_id)
);

-- Arena queue
CREATE TABLE arena_queue (
    character_id INT PRIMARY KEY,
    queue_type VARCHAR(20),
    elo_rating INT,
    queued_at TIMESTAMP
);
```

**Testing:** Create tables, insert test data, verify queries

#### 1.10: Configuration Framework
**Priority:** 🟡 HIGH

**New Config File:** `config/arena.ini`

```ini
[Arena]
Enabled = yes
MinLevel = 10              # Minimum level to enter

[Matchmaking]
QueueTimeout = 300         # 5 minutes max wait
ELORange = 200            # ±200 ELO matching
StartingELO = 1200        # New player ELO

[Match]
TimeLimit = 600           # 10 minute time limit
RespawnDelay = 5          # Seconds before respawn
KillLimit = 5             # First to N kills wins
ActivityTimeout = 120     # 2 minutes no combat = draw

[Rewards]
WinELO = 25              # Base ELO gain
LossELO = 15             # Base ELO loss
WinPoints = 100          # Arena currency
LossPoints = 10          # Participation reward
```

**Testing:** Load config, validate values, test overrides

### Deliverables

- [ ] 4 critical bugs fixed
- [ ] 4 P0 unknowns researched and documented
- [ ] Database schema created and tested
- [ ] Arena configuration framework
- [ ] Test suite for critical paths
- [ ] Updated documentation with findings

### Success Criteria

- [ ] All P0 bugs fixed (no crashes, no corruption)
- [ ] Research questions answered (update UNKNOWN-BEHAVIOR.md)
- [ ] Database queries functional (<100ms for rankings)
- [ ] Config system loads and validates
- [ ] All tests passing

### Risk Assessment

**Medium Risk:**
- Bug fixes may introduce new bugs (mitigation: comprehensive testing)
- Research may reveal blockers (mitigation: alternative approaches documented)
- Database performance unknown (mitigation: benchmarking early)

---

## Phase 2: Core Arena Features

**Status:** 🔜 FUTURE  
**Duration:** TBD (entry/exit criteria below)  
**Goal:** Implement basic matchmaking and 1v1 arena

### Prerequisites
- Phase 1 complete
- Critical bugs fixed
- Research findings documented

### Core Systems

#### 2.1: Queue System
**Priority:** ⭐⭐⭐⭐⭐

**Features:**
- `/arena queue` command (or NPC interaction)
- Queue join/leave
- ELO-based matchmaking
- Queue timeout (5 minutes)
- Match found notification

**Packets:**
- `MESSAGE/OPEN` for notifications
- `QUEST/DIALOG` for queue UI (if virtual NPCs work)

**Testing:**
- Solo queue → timeout
- Two players queue → matched
- ELO range filtering

#### 2.2: Match Lifecycle Manager
**Priority:** ⭐⭐⭐⭐⭐

**Features:**
- Match creation
- Player teleportation
- Match state tracking
- Time limit enforcement
- Kill limit detection
- Victory condition checking

**Classes:**
```cpp
class ArenaMatch {
    int match_id;
    MatchType type;
    std::vector<Character*> participants;
    std::map<Character*, int> kills;
    double start_time;
    double time_limit;
    MatchState state;
    
    void Start();
    void HandleKill(Character* killer, Character* victim);
    void CheckVictoryCondition();
    void End(Character* winner);
};
```

**Testing:**
- Full match lifecycle
- Time limit expiration
- Kill limit reached
- Player disconnect

#### 2.3: Combat System Overhaul
**Priority:** ⭐⭐⭐⭐⭐

**Features:**
- Replace instant-elimination with HP-based combat
- Track damage dealt/taken
- Respawn system
- Kill/death tracking
- Combat log generation

**Integration:**
- Hook `Map::Attack()` and `Map::SpellAttack()`
- Check `character->arena_match != NULL`
- Route through match combat handler

**Testing:**
- Attack reduces HP
- HP=0 → death → respawn
- Kill count increments
- Match ends at kill limit

#### 2.4: Basic Match UI
**Priority:** ⭐⭐⭐⭐

**Features:**
- Match start notification (INFO BOX)
- Kill notifications (SERVER CHAT)
- Match end report (INFO BOX)

**Example:**
```
[INFO BOX] "Match Started"
"1v1 Deathmatch - First to 5 Kills"
"Opponent: PlayerName (1850 ELO)"
"Good luck!"

[CHAT] "Server  PlayerA eliminated PlayerB! (3-1)"

[INFO BOX] "Victory!"
"Winner: PlayerA (5-3)"
"ELO: 1825 → 1843 (+18)"
"Arena Points: +100"
"Duration: 4:32"
```

**Testing:**
- UI displays at correct times
- Data accuracy
- Client rendering

#### 2.5: Basic ELO System
**Priority:** ⭐⭐⭐⭐

**Formula:**
```
Expected Score = 1 / (1 + 10^((OpponentELO - PlayerELO) / 400))
ELO Change = K * (Actual - Expected)

K-Factor:
  - New players (< 30 matches): K=40
  - Active players: K=20
  - Established players (> 100 matches): K=10
```

**Features:**
- Calculate ELO change on match end
- Update database
- Display to players

**Testing:**
- Similar ELO → ~equal change
- Higher ELO wins → small gain
- Lower ELO wins → large gain
- Upset victories

### Deliverables

- [ ] Queue system functional
- [ ] Match manager handles full lifecycle
- [ ] HP-based combat system
- [ ] Match UI (notifications, reports)
- [ ] ELO calculation and persistence
- [ ] 1v1 arena fully playable

### Success Criteria

- [ ] Two players can queue, match, and fight
- [ ] Matches end correctly (time/kill limit)
- [ ] ELO updates accurately
- [ ] UI displays all relevant information
- [ ] No crashes or desyncs during matches

### Risk Assessment

**Medium Risk:**
- Combat system redesign complex (mitigation: incremental testing)
- Match state management tricky (mitigation: state machine diagram)
- ELO formula tuning needed (mitigation: adjustable K-factor)

---

## Phase 3: Statistics & Persistence

**Status:** 🔜 FUTURE  
**Duration:** TBD (entry/exit criteria below)  
**Goal:** Track detailed stats, create leaderboards, match history

### Prerequisites
- Phase 2 complete
- 1v1 arena functional
- Database schema validated

### Features

#### 3.1: Detailed Statistics Tracking
**Priority:** ⭐⭐⭐⭐

**Stats:**
- Total matches, wins, losses, win rate
- Kills, deaths, K/D ratio
- Damage dealt, damage taken
- Average match time, fastest win
- Current streak, best streak
- ELO history (peak, current)

**Storage:** Expand `arena_players` table

#### 3.2: Leaderboard System
**Priority:** ⭐⭐⭐⭐⭐

**Implementation:** MESSAGE BOARD

**Features:**
- Top 100 by ELO
- Top 100 by wins
- Top 100 by K/D ratio
- Weekly/monthly leaderboards

**Board Format:**
```
Post 1: "#1 - PlayerName (2150 ELO)" 
Body: Full player stats

Post 2: "#2 - OtherPlayer (2080 ELO)"
Body: Full player stats
```

**Updates:**
- Regenerate board on match end
- Limit to top 100 (performance)
- Cache results (5 minute refresh)

**Testing:**
- Board displays correctly
- Clicking shows detailed stats
- Updates after matches
- Performance with 1000+ players

#### 3.3: Match History System
**Priority:** ⭐⭐⭐⭐

**Implementation:** MESSAGE BOARD (per-player) or INFO BOX (query-based)

**Features:**
- Last 50 matches per player
- Opponent name, result, ELO change
- Date, duration, kills
- Click for detailed match stats

**Board Format:**
```
Post 1: "Victory vs PlayerName"
Body: Match #12345 detailed stats

Post 2: "Defeat vs OtherPlayer"
Body: Match #12346 detailed stats
```

**Testing:**
- History accurate
- Pagination works
- Performance acceptable

#### 3.4: Player Profile System
**Priority:** ⭐⭐⭐

**Implementation:** INFO BOX or BOOK (character info)

**Features:**
- View any player's arena stats
- Compare stats
- Arena achievements

**Command:** `/arena stats PlayerName`

**Display:**
```
=== PlayerName's Arena Profile ===

Rank: #42 (1850 ELO)
Tier: Gold III

Total Matches: 120
Wins: 75 (62.5%)
Losses: 45 (37.5%)

K/D Ratio: 1.8 (450 kills, 250 deaths)
Average Match: 4:35
Fastest Win: 1:23

Current Streak: 3 wins
Best Streak: 12 wins

Peak ELO: 1920 (3 days ago)
```

**Testing:**
- Stats display correctly
- Data matches database
- Performance acceptable

### Deliverables

- [ ] Comprehensive stat tracking
- [ ] Leaderboard boards created
- [ ] Match history accessible
- [ ] Player profile viewer
- [ ] Stat persistence verified

### Success Criteria

- [ ] All stats accurate
- [ ] Leaderboards update correctly
- [ ] History displays last 50 matches
- [ ] Profile queries <100ms
- [ ] No database bottlenecks

---

## Phase 4: Competitive Features

**Status:** 🔜 FUTURE  
**Duration:** TBD (entry/exit criteria below)  
**Goal:** Add ranked seasons, rewards, achievements

### Prerequisites
- Phase 3 complete
- Statistics system functional
- Large player base (for testing)

### Features

#### 4.1: Ranked Seasons
**Priority:** ⭐⭐⭐⭐

**Features:**
- Season duration (config-based)
- ELO reset between seasons
- Season-specific leaderboards
- End-of-season rewards

**Schema:**
```sql
CREATE TABLE arena_seasons (
    season_id INT PRIMARY KEY,
    season_name VARCHAR(50),
    start_date TIMESTAMP,
    end_date TIMESTAMP,
    active BOOLEAN
);

CREATE TABLE arena_season_stats (
    season_id INT,
    character_id INT,
    elo_rating INT,
    -- ... other stats
    PRIMARY KEY (season_id, character_id)
);
```

#### 4.2: Reward Shop
**Priority:** ⭐⭐⭐⭐

**Implementation:** SHOP system

**Features:**
- Arena Points currency (earned from matches)
- Trophy items (cosmetic)
- Exclusive equipment
- Titles

**Items (EIF):**
- Arena Token (ID 5000) - currency
- Bronze Trophy (ID 5001) - 100 points
- Silver Trophy (ID 5002) - 500 points
- Gold Trophy (ID 5003) - 1000 points
- Arena Blade (ID 5010) - 5000 points

**NPC (ENF):**
- Arena Vendor (ID 501) - Shop type

**Testing:**
- Purchase works
- Currency deducted
- Items granted
- Shop UI displays correctly

#### 4.3: Achievement System
**Priority:** ⭐⭐⭐

**Implementation:** QUEST system (quest list as achievements)

**Achievements:**
- First Victory
- 10 Wins
- 50 Wins
- 100 Wins
- First Blood Master (5x first blood)
- Untouchable (10 wins, 0 deaths)
- Comeback King (win from 0-4 deficit)
- Flawless Victory (5-0 win)

**Rewards:**
- Arena Points
- Titles
- Trophy items

**Testing:**
- Achievements trigger correctly
- Rewards granted
- Progress tracked

#### 4.4: Ranking Tiers
**Priority:** ⭐⭐⭐

**Tiers:**
```
Unranked: < 1000 ELO
Bronze:   1000-1199
Silver:   1200-1399
Gold:     1400-1599
Platinum: 1600-1799
Diamond:  1800-1999
Master:   2000-2199
Grandmaster: 2200+
```

**Features:**
- Tier displayed in profile
- Tier-specific rewards
- Promotion/demotion notifications

**Testing:**
- Tiers calculate correctly
- Promotions trigger
- Rewards granted

### Deliverables

- [ ] Ranked seasons system
- [ ] Reward shop operational
- [ ] Achievement system
- [ ] Ranking tiers implemented

### Success Criteria

- [ ] Seasons start/end cleanly
- [ ] Rewards purchasable
- [ ] Achievements unlock correctly
- [ ] Tiers display accurately

---

## Phase 5: Advanced Features

**Status:** 🔜 FUTURE  
**Duration:** TBD (entry/exit criteria below)  
**Goal:** Team modes, tournaments, spectators (if possible)

### Prerequisites
- Phase 4 complete
- Stable competitive environment
- Large active player base

### Features

#### 5.1: Team Arena Modes (2v2, 3v3)
**Priority:** ⭐⭐⭐⭐⭐  
**Dependency:** Party system research (Q4.3)

**Features:**
- Party-based queue
- Team ELO calculation
- Team leaderboards
- Friendly fire off

**Testing:**
- Party queuing
- Team combat
- Victory conditions
- ELO distribution

#### 5.2: Free-For-All (FFA) Mode
**Priority:** ⭐⭐⭐⭐

**Features:**
- 4-8 players
- Last standing wins
- Points for kills
- Survival time tracked

**Testing:**
- Handles 8 players
- Performance acceptable
- Victory conditions correct

#### 5.3: Tournament System
**Priority:** ⭐⭐⭐

**Features:**
- Bracket generation
- Single/double elimination
- Best-of-N matches
- Tournament admin tools

**UI:**
- Message board for bracket display
- Quest dialog for registration

**Testing:**
- Brackets generate correctly
- Matches progress
- Winners determined

#### 5.4: Spectator System (Experimental)
**Priority:** ⭐⭐  
**Dependency:** Research required

**Features:**
- Spectator map entry
- Observer mode (no combat)
- Match viewing

**Challenges:**
- Client has no spectator mode
- Would need creative workarounds
- Low priority / high effort

### Deliverables

- [ ] Team modes functional
- [ ] FFA mode functional
- [ ] Tournament system
- [ ] Spectator system (if feasible)

### Success Criteria

- [ ] Team matches work smoothly
- [ ] FFA supports 8 players
- [ ] Tournaments complete successfully
- [ ] Spectators can observe (if implemented)

---

## Feature Prioritization Matrix

### Must-Have (Phase 1-2)
- ✅ Critical bug fixes
- ✅ Queue system
- ✅ Match lifecycle
- ✅ HP-based combat
- ✅ Basic ELO
- ✅ Match UI

### Should-Have (Phase 3-4)
- 📊 Statistics tracking
- 📊 Leaderboards
- 📊 Match history
- 🏆 Ranked seasons
- 🏆 Reward shop
- 🏆 Achievements

### Could-Have (Phase 5)
- 🌟 Team modes
- 🌟 FFA mode
- 🌟 Tournaments

### Won't-Have (Client Required)
- ❌ Custom UI overlays
- ❌ Real-time health bars
- ❌ Replay system
- ❌ Built-in spectator camera

---

## Success Metrics

### Phase 1
- ⚠️ Zero critical bugs remaining
- ✅ All P0 research questions answered
- ✅ Database schema validated

### Phase 2
- ✅ 100 successful matches without crash
- ✅ Average match duration within acceptable range
- ✅ ELO calculations accurate

### Phase 3
- ✅ Leaderboards update within 5 minutes
- ✅ Stat queries <100ms
- ✅ 10,000+ matches in history

### Phase 4
- ✅ Season transitions cleanly
- ✅ Rewards purchased by players
- ✅ Achievements earned

### Phase 5
- ✅ Team matches functional
- ✅ Tournaments complete
- ✅ Community engagement high

---

## Risk Management

### High Risks
1. **Party system incompatible** → Blocks team modes
   - *Mitigation:* Research in Phase 1, plan alternatives
2. **Performance bottlenecks** → Lag/crashes
   - *Mitigation:* Benchmark early, optimize queries
3. **Client limitations discovered** → Blocks features
   - *Mitigation:* Creative workarounds, alternative approaches

### Medium Risks
1. **Database schema changes** → Data migration
   - *Mitigation:* Design extensible schema upfront
2. **ELO formula needs tuning** → Player dissatisfaction
   - *Mitigation:* Configurable K-factor, seasonal adjustments
3. **Bug fixes introduce regressions** → New bugs
   - *Mitigation:* Comprehensive test suite

### Low Risks
1. **UI polish lacking** → Poor UX
   - *Mitigation:* Iterative improvement, player feedback
2. **Achievement balance** → Too easy/hard
   - *Mitigation:* Adjustable thresholds

---

## Maintenance & Support

### Ongoing Tasks (Post-Launch)
- Bug fixes as discovered
- Performance optimization
- ELO formula tuning
- New achievements
- Seasonal content
- Balance adjustments

### Documentation Updates
- Update docs as features complete
- Document new findings
- Maintain changelog
- Update API references

### Community Engagement
- Gather feedback
- Monitor balance
- Address exploits
- Feature requests

---

## Conclusion

This roadmap provides a structured, evidence-based path to building modern arena features on ArenaServ. By fixing critical bugs first, researching unknowns early, and delivering incremental value, we minimize risk and maximize player benefit.

**Next Step:** Begin Phase 1 - Foundation & Bug Fixes

**Timeline:** Milestone-based progression
**Risk Level:** Medium (mitigated by phased approach and research)  
**Success Likelihood:** High (based on Phase 0 findings)

---

**Last Review:** 2026-09-22  
**Next Review:** After Phase 1 completion


