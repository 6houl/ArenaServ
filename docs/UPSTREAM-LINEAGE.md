# ArenaServ Upstream Lineage

**Status:** CONFIRMED  
**Last Updated:** 2026-09-22

## Project History

```
EOSERV (2006-2014)
│   Original author: Sausage
│   License: zlib
│   Repository: https://github.com/eoserv/eoserv
│   Language: C++
│   Purpose: Open-source Endless Online server
│
└── Etheos (2014-present)
    │   Fork maintainer: ethanmoffat
    │   Repository: https://github.com/ethanmoffat/etheos
    │   Renamed from EOSERV → Etheos
    │   Active development and bug fixes
    │   Public test server: etheos.moffat.io:8078
    │
    └── ArenaServ (2026-present)
        │   Purpose: Arena/minigame-focused server
        │   Focus: Enhanced PvP systems
        │   Constraint: Vanilla client compatibility
```

## EOSERV (Original)

**Created:** ~2006  
**Author:** Sausage  
**Status:** Historical reference

### Key Contributions
- First open-source EO server implementation
- Reverse-engineered EO protocol
- Established C++ codebase architecture
- Basic arena system implementation
- Quest system (EO+)
- Guild system
- Wedding system
- PK system

**Evidence Sources:**
- `references/` - Not present in ArenaServ
- GitHub: https://github.com/eoserv/eoserv (historical)
- License preserved in `LICENSE.txt`

## Etheos (Direct Upstream)

**Forked:** ~2014  
**Maintainer:** ethanmoffat  
**Repository:** https://github.com/ethanmoffat/etheos  
**Status:** ACTIVE, reference implementation

### Major Changes from EOSERV
- Renamed project
- CMake modernization
- C++14/C++17 adoption
- CI/CD with GitHub Actions
- Docker support
- SQL Server support added
- Bug fixes and stability improvements
- Integration testing with EOBot
- Public test server deployment

### Etheos Features
- All EOSERV features
- Improved build system
- Better database abstraction
- Enhanced configuration system
- More robust error handling
- Active bug fixing

**Evidence Sources:**
- `README.md` - References etheos GitHub
- `CMakeLists.txt` - Project name "etheos"
- `.github/workflows/` - Etheos CI configuration
- `Dockerfile` - etheos Docker image
- Build scripts reference etheos

### Etheos as Reference
ArenaServ can use Etheos for:
- Protocol behavior verification
- Bug fix backporting
- Feature comparison
- Live server testing (etheos.moffat.io:8078)

## ArenaServ (Current Project)

**Created:** 2026  
**Focus:** Arena/minigame enhancement  
**Repository:** Local (not yet published)  
**Status:** ACTIVE DEVELOPMENT

### Relationship to Etheos
- **Downstream fork** of Etheos
- Maintains protocol compatibility
- Can backport Etheos bug fixes
- Diverges in arena/minigame features
- Still references "etheos" in build artifacts

**Evidence:**
```
CMakeLists.txt:9: project(etheos VERSION 0.7.1 LANGUAGES C CXX)
```

### ArenaServ Differentiation
1. **Enhanced Arena System**
   - Beyond legacy implementation
   - Statistics tracking
   - Leaderboards
   - ELO ranking
   - Rewards system
   - Multiple match types

2. **Server-Side Innovation**
   - Work within vanilla client
   - Creative use of existing packets
   - Data-driven features
   - No client modifications required

3. **PvP Focus**
   - Competitive features
   - Match making
   - Tournaments
   - Ladder system

## Reference Materials in Repository

### references/Client/
- **Status:** CONFIRMED present
- **Contents:** Vanilla Endless Online client
- **Files:** Endless.exe, EConfig.exe, data files, graphics
- **Purpose:** Testing and protocol verification

### references/eo-protocol-master/
- **Status:** CONFIRMED present
- **Contents:** Modern EO protocol specification
- **Format:** XML protocol definitions
- **Purpose:** Protocol reference and validation

### references/EOSource-Revived-main/
- **Status:** CONFIRMED present
- **Contents:** Alternative EOSERV-based implementation
- **Purpose:** Cross-reference and comparison

## Version Identification

### Current Build Version
```
Version: 0.7.1
Project Name: etheos (inherited)
```

**Evidence:** `CMakeLists.txt:9`

### Recommended Future Versioning
```
ArenaServ 1.0.0 (based on Etheos 0.7.1)
```

## License Inheritance

**License:** zlib License  
**Origin:** EOSERV  
**File:** `LICENSE.txt`

### License Requirements
- Attribution preserved
- Modification allowed
- Commercial use allowed
- Source distribution not required
- Must include copyright notice

**Compliance Status:** ✅ CONFIRMED
- Original EOSERV attribution present
- License file included
- Copyright notices maintained

## Upstream Synchronization Strategy

### Pulling from Etheos
1. Monitor https://github.com/ethanmoffat/etheos
2. Review bug fixes for applicability
3. Cherry-pick or merge relevant changes
4. Test against ArenaServ modifications
5. Maintain divergent arena features

### Contributing Back to Etheos
- General bug fixes can be upstreamed
- Arena-specific features kept in ArenaServ
- Protocol improvements shareable
- Build system improvements shareable

## Related Projects

### EOLib
- **Purpose:** EO protocol library
- **Language:** C#
- **Relation:** Different ecosystem, reference only
- **Use:** Protocol specification validation

### EndlessClient
- **Purpose:** Cross-platform EO client
- **Maintainer:** ethanmoffat (same as Etheos)
- **Relation:** Client counterpart
- **Use:** Protocol testing

### EOBot
- **Purpose:** Automated client testing
- **Use:** Integration testing
- **Relation:** Used in Etheos CI

## Source Code Attribution

Key files inherited from EOSERV/Etheos:
- All `src/*.cpp` and `src/*.hpp` files
- `cmake/` build system
- Handler implementations
- Packet processing
- Database layer
- Configuration system
- Timer system
- Map loading
- Character management
- Combat system
- Guild system
- Quest system (EO+)
- Wedding system
- **Arena system (legacy)**

ArenaServ modifications:
- Enhanced arena documentation (this docs/ folder)
- Future arena enhancements (to be implemented)
- ArenaServ-specific features (planned)

---

**Key Takeaway:** ArenaServ inherits a mature, stable codebase from EOSERV → Etheos. The core server functionality is proven. Arena enhancements are the primary differentiation.
