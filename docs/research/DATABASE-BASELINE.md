# Database Baseline (Phase 0.5)

**Source:** Etheos v0.7.1  
**Schema:** `install.sql`  
**Date:** 2026-09-22

## Database Tables

### Core Tables (Arena-Relevant)

#### characters
**Purpose:** Player persistent state  
**Arena Fields:** NONE  
**Evidence:** `install.sql:19-72`

**Key Fields:**
- `name` (PK)
- `account`
- `class`, `level`, `exp`
- `hp`, `tp`, stats
- `map`, `x`, `y`
- No arena-specific columns

**Finding:** Legacy arena stores NO persistent data.

Arena state is entirely runtime (`Character::arena`, `Character::arena_kills`).

---

#### accounts
**Purpose:** Account authentication  
**Arena Relevance:** None  
**Evidence:** `install.sql:2-16`

---

#### guilds
**Purpose:** Guild management  
**Arena Relevance:** Future (guild arena matches)  
**Evidence:** `install.sql:74-83`

---

#### bans
**Purpose:** Ban management  
**Arena Relevance:** None  
**Evidence:** `install.sql:85-93`

---

#### reports
**Purpose:** Player reports  
**Arena Relevance:** Future (arena misconduct reporting)  
**Evidence:** `install.sql:95-101`

---

## Arena Persistence

### Current State: NONE

The legacy arena system stores **ZERO** persistent data:
- No match history
- No statistics
- No rankings
- No player records
- No seasonal data

All state is runtime-only:
- `Character::arena` (current arena pointer)
- `Character::arena_kills` (current match kills)
- `Character::next_arena` (arena to join)
- `Arena::occupants` (current player count)

### Evidence

`src/character.hpp:205-207`:
```cpp
char arena_kills;     // Runtime only
Arena *arena;         // Runtime pointer
Arena *next_arena;    // Runtime pointer
```

No `CREATE TABLE arena_*` found in:
- `install.sql`
- `install_sqlserver.sql`
- `upgrade/*.sql`

### Implication for ArenaServ

ArenaServ will need NEW tables for:
- Match history
- Player statistics
- Rankings/leaderboards
- Seasonal records
- Rewards tracking

**All new schema - not modifying legacy.**

---

## Schema Version

**Current:** Etheos v0.7.1  
**Migration Path:** `upgrade/0.7.0_to_0.7.1.sql`

No arena-related schema in migrations.

---

## Database Backend Support

**Supported:**
- MySQL/MariaDB (`config/database/mysql.ini`)
- SQLite (`config/database/sqlite.ini`)
- SQL Server (`config/database/sqlserver.ini`)

**Schema Files:**
- MySQL: `install.sql`
- SQL Server: `install_sqlserver.sql`

ArenaServ must provide schema for all three backends.

---

## Verdict

Legacy arena has **zero** persistence. Entirely runtime state.

ArenaServ requires **new database design** from scratch.
