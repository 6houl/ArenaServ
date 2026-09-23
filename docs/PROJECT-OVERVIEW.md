# ArenaServ Project Overview

**Status:** CONFIRMED  
**Last Updated:** 2026-09-22

## Project Identity

**Name:** ArenaServ  
**Type:** Endless Online Game Server  
**Language:** C++14/C++17  
**Platform:** Cross-platform (Windows, Linux)  
**License:** zlib (inherited from EOSERV)

## Project Ancestry

```
EOSERV (Original, by Sausage)
    ↓
Etheos (Fork by ethanmoffat)
    ↓
ArenaServ (Current project)
```

See [UPSTREAM-LINEAGE.md](UPSTREAM-LINEAGE.md) for detailed history.

## Core Purpose

ArenaServ is a custom Endless Online server implementation focused on:

1. **PvP Arena System** - Enhanced arena/minigame framework
2. **Stability** - Bug fixes and reliability improvements
3. **Extensibility** - Server-side feature enhancements within vanilla client constraints
4. **Compatibility** - Works with unmodified Endless Online clients

## Key Constraint

**The vanilla Endless Online client is fixed and cannot be modified.**

All features must work within:
- Existing protocol packets
- Existing UI windows
- Existing data file formats
- Existing client behavior

## Current State

### Build System
- **CMake** 3.5+ (cross-platform)
- **Visual Studio 2017-2022** (Windows)
- **GCC 4.8+** or **Clang 3.4+** (Linux)

### Database Support
- Microsoft SQL Server (ODBC)
- MariaDB/MySQL
- SQLite3

### Major Components

1. **World** - Global server state, player management
2. **Map** - Individual map instances, NPC management
3. **Character** - Player character state and actions
4. **Arena** - PvP arena system (legacy implementation)
5. **Quest** - EO+ quest system
6. **Guild** - Guild management
7. **Party** - Party system
8. **NPC** - NPC AI and interactions
9. **Shop** - Commerce system
10. **Board** - Message board system
11. **Wedding** - Marriage system
12. **Combat** - PvM and PvP combat
13. **Timer** - Event scheduling system
14. **Packet** - Network protocol handling
15. **Database** - Persistence layer
16. **Config** - Configuration management

### Data Files

- **EIF** - Items (equipment, consumables, etc.)
- **ENF** - NPCs (monsters, vendors, etc.)
- **ESF** - Spells/skills
- **ECF** - Classes
- **EMF** - Maps (tiles, warps, NPCs, chests)

### Configuration Files

- **config.ini** - Main server configuration
- **admin.ini** - Admin/command permissions
- **config/server.ini** - Network settings
- **config/database.ini** - Database connection
- **config/rates.ini** - EXP/drop rates
- **config/pk.ini** - PK system settings
- **config/guilds.ini** - Guild settings
- **config/misc.ini** - Miscellaneous settings
- **data/arenas.ini** - Arena configuration
- **data/drops.ini** - Drop tables
- **data/shops.ini** - Shop inventories
- **data/formulas.ini** - Damage/stat formulas
- **data/home.ini** - Starting locations
- **data/skills.ini** - Skill configuration
- **data/speech.ini** - NPC speech patterns

## Network Protocol

- **Protocol Family:** Endless Online Protocol
- **Packet Format:** Family + Action + Data
- **Encoding:** Custom integer encoding
- **Encryption:** Interleaving + key exchange

### Key Packet Families (relevant to ArenaServ)

- PACKET_ARENA (45) - Arena-specific packets
- PACKET_MESSAGE (35) - Status/info messages
- PACKET_BOARD (43) - Message boards
- PACKET_QUEST (50) - Quest dialogs
- PACKET_SHOP (13) - Shop interfaces
- PACKET_BOOK (51) - Character info display
- PACKET_PAPERDOLL (30) - Equipment display
- PACKET_NPC (26) - NPC interactions
- PACKET_ATTACK (11) - Combat
- PACKET_WALK (6) - Movement
- PACKET_WARP (19) - Map transitions

## Legacy Arena System

**Status:** CONFIRMED (implemented, functional, has limitations)

The current arena system provides:
- Timed spawning from source tiles to destination tiles
- PvP combat detection and elimination
- Kill tracking
- Match-end detection (when 2 players remain, winner warps out)
- Capacity blocking

See [arena/](arena/) directory for complete documentation.

## Development Philosophy

### What ArenaServ IS
- A feature-enhanced EOSERV fork
- Compatible with vanilla clients
- Data-driven where possible
- Server-side innovation

### What ArenaServ IS NOT
- A client modification project
- Breaking protocol compatibility
- Requiring custom clients
- Abandoning vanilla client support

## Testing Approach

1. **Unit Tests** - GoogleTest framework
2. **Integration Tests** - Database, file I/O, timer system
3. **Manual Tests** - Vanilla client testing required for UI/protocol
4. **Reference Implementation** - Etheos public server for behavior verification

## Current Known Issues

See [research/UNKNOWN-BEHAVIOR.md](research/UNKNOWN-BEHAVIOR.md) for:
- Unconfirmed client capabilities
- Protocol edge cases
- Reentrancy issues in legacy systems
- State synchronization concerns

## Future Vision

ArenaServ aims to become:
1. Most stable EOSERV-based server
2. Reference implementation for advanced arena/minigame systems
3. Platform for competitive PvP within vanilla client constraints
4. Well-documented codebase for future developers

---

**Sources:**
- `CMakeLists.txt` - Build configuration
- `README.md` - Project readme
- `src/` - Source code structure
- `config/` - Configuration files
- `data/` - Data file templates

