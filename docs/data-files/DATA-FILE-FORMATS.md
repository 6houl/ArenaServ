# EO Data File Formats

**Status:** CONFIRMED (ArenaServ source code analysis)  
**Last Updated:** 2026-09-22  
**Purpose:** Complete reference for all EO binary data formats

## Overview

Endless Online uses binary "pub" files to store game data synchronized between server and client:

| File | Extension | Purpose | Server-Side Editable |
|------|-----------|---------|---------------------|
| Item File | `.eif` | Item definitions | ✅ Yes |
| NPC File | `.enf` | NPC definitions | ✅ Yes |
| Spell File | `.esf` | Spell/skill definitions | ✅ Yes |
| Class File | `.ecf` | Class definitions | ✅ Yes |
| Map File | `.emf` | Map data (per-map) | ⚠️ Via map editor |

**Evidence:** `src/eodata.hpp`, `data/pub/`

---

## File Distribution

### Server → Client
When a player connects, the server sends its pub files to the client. The client **must** use server-provided files.

**Process:**
1. Client connects
2. Server sends EIF, ENF, ESF, ECF via `PACKET_F_INIT`
3. Client caches files locally
4. Map (EMF) sent when player warps via `PACKET_WARP`

**Implication:** Server can define **custom items, NPCs, spells, classes** and they will work with vanilla client.

---

## Common File Structure

All pub files share a common header format:

```
Offset | Size | Type          | Field
-------|------|---------------|------------------
0x00   | 3    | char[3]       | Magic ("EIF", "ENF", "ESF", "ECF", "EMF")
0x03   | 4    | unsigned char | RID (revision ID, arbitrary)
0x07   | 2    | unsigned char | Length (number of records)
0x09   | 1    | unsigned char | Version (always 1)
0x0A   | ... | ...            | Data records
```

**Evidence:** `src/eodata.hpp`, read functions in `src/eodata.cpp`

### RID (Revision ID)
- 4-byte arbitrary identifier
- Server and client RIDs must match
- Changed when file updated to force client redownload
- Not a version number, just a unique ID

### Encoding
- **Numbers:** Encoded using `PacketProcessor::Number()` - EO-specific encoding
- **Strings:** Break-string format (0xFF terminated)
- **Byte order:** Little-endian style encoding

**Evidence:** `src/packet.cpp` (PacketProcessor)

---

## 1. EIF - Item File Format

**Purpose:** Defines all items (equipment, consumables, keys, etc.)  
**Location:** `data/pub/*.eif`  
**Record Size:** 58 bytes per item

### Item Types

```cpp
enum Type {
    Static = 0,       // Non-interactive item
    UnknownType1,
    Money,            // Gold currency
    Heal,             // HP/TP healing item
    Teleport,         // Warp scroll
    Spell,            // Spell scroll (learn spell)
    EXPReward,        // EXP granting item
    StatReward,       // Stat boost item
    SkillReward,      // Skill point item
    Key,              // Key (opens doors/chests)
    Weapon,           // Weapon (equippable)
    Shield,           // Shield (equippable)
    Armor,            // Chest armor (equippable)
    Hat,              // Hat (equippable)
    Boots,            // Boots (equippable)
    Gloves,           // Gloves (equippable)
    Accessory,        // Accessory (equippable)
    Belt,             // Belt (equippable)
    Necklace,         // Necklace (equippable)
    Ring,             // Ring (equippable)
    Armlet,           // Armlet (equippable)
    Bracer,           // Bracer (equippable)
    Beer,             // Alcohol item
    EffectPotion,     // Visual effect potion
    HairDye,          // Hair color change
    CureCurse         // Remove curse
};
```

**Evidence:** `src/eodata.hpp:99-127`

### Item SubTypes

```cpp
enum SubType {
    None = 0,
    Ranged,       // Ranged weapon (bow)
    Arrows,       // Ammunition
    Wings,        // Wings equipment
    TwoHanded     // Two-handed weapon
};
```

### Item Special Flags

```cpp
enum Special {
    Normal = 0,
    Rare,         // Rare item (?)
    UnknownSpecial2,
    Unique,       // Unique item (?)
    Lore,         // Lore item (cannot drop/trade)
    Cursed        // Cursed item
};
```

### Item Sizes

```cpp
enum Size {
    Size1x1 = 0,  // 1 inventory slot
    Size1x2,      // 2 slots vertical
    Size1x3,      // 3 slots vertical
    Size1x4,      // 4 slots vertical
    Size2x1,      // 2 slots horizontal
    Size2x2,      // 4 slots (2x2 grid)
    Size2x3,      // 6 slots (2x3 grid)
    Size2x4       // 8 slots (2x4 grid)
};
```

### EIF Data Structure

```cpp
struct EIF_Data {
    int id;                    // Item ID (unique)
    std::string name;          // Item name (displayed)
    short graphic;             // Graphic ID (GFX file reference)
    
    Type type;                 // Item type (weapon, armor, etc.)
    SubType subtype;           // Item subtype
    Special special;           // Special flags
    
    // Stats
    short hp;                  // HP bonus
    short tp;                  // TP bonus
    short mindam;              // Minimum damage
    short maxdam;              // Maximum damage
    short accuracy;            // Accuracy bonus
    short evade;               // Evade bonus
    short armor;               // Armor value
    
    // Stat bonuses
    unsigned char str;         // Strength
    unsigned char intl;        // Intelligence
    unsigned char wis;         // Wisdom
    unsigned char agi;         // Agility
    unsigned char con;         // Constitution
    unsigned char cha;         // Charisma
    
    // Elemental modifiers
    unsigned char light;
    unsigned char dark;
    unsigned char earth;
    unsigned char air;
    unsigned char water;
    unsigned char fire;
    
    // Type-specific data (union)
    union {
        int scrollmap;         // Teleport: destination map
        int dollgraphic;       // Equipment: paperdoll graphic
        int expreward;         // EXPReward: experience amount
        int haircolor;         // HairDye: color ID
        int effect;            // EffectPotion: effect ID
        int key;               // Key: key type
    };
    
    union {
        unsigned char gender;  // Equipment: gender restriction (0=all, 1=male, 2=female)
        unsigned char scrollx; // Teleport: X coordinate
    };
    
    union {
        unsigned char scrolly; // Teleport: Y coordinate
        unsigned char dual_wield_dollgraphic; // Weapon: offhand graphic
    };
    
    // Requirements
    short levelreq;            // Level requirement
    short classreq;            // Class requirement (0=all)
    short strreq;              // Strength requirement
    short intreq;              // Intelligence requirement
    short wisreq;              // Wisdom requirement
    short agireq;              // Agility requirement
    short conreq;              // Constitution requirement
    short chareq;              // Charisma requirement
    
    unsigned char weight;      // Item weight
    Size size;                 // Inventory size
};
```

**Evidence:** `src/eodata.hpp:17-93`

### Arena Applications

#### Arena Tokens/Currency
```cpp
// Custom currency for arena rewards
EIF_Data arena_token;
arena_token.id = 5000;
arena_token.name = "Arena Token";
arena_token.type = EIF::Static;  // or Money for tradeable currency
arena_token.special = EIF::Lore;  // Cannot drop/trade
arena_token.graphic = 100;        // Gold coin graphic
arena_token.size = EIF::Size1x1;
```

#### Trophy Items
```cpp
// Cosmetic trophy for achievements
EIF_Data trophy_item;
trophy_item.id = 5001;
trophy_item.name = "Arena Champion Trophy";
trophy_item.type = EIF::Static;
trophy_item.special = EIF::Unique;
trophy_item.graphic = 150;  // Trophy graphic
trophy_item.size = EIF::Size1x1;
```

#### Arena Rewards
```cpp
// Special equipment as reward
EIF_Data arena_sword;
arena_sword.id = 5002;
arena_sword.name = "Arena Master's Blade";
arena_sword.type = EIF::Weapon;
arena_sword.graphic = 200;
arena_sword.mindam = 50;
arena_sword.maxdam = 100;
arena_sword.levelreq = 50;
arena_sword.size = EIF::Size1x2;
```

---

## 2. ENF - NPC File Format

**Purpose:** Defines all NPCs (monsters, vendors, quest NPCs)  
**Location:** `data/pub/*.enf`  
**Record Size:** 39 bytes per NPC

### NPC Types

```cpp
enum Type {
    NPC = 0,       // Generic NPC
    Passive,       // Passive monster (doesn't attack)
    Aggressive,    // Aggressive monster (attacks on sight)
    Unknown1,
    Unknown2,
    Unknown3,
    Shop,          // Shop vendor
    Inn,           // Inn keeper
    Unknown4,
    Bank,          // Banker
    Barber,        // Barber
    Guild,         // Guild master
    Priest,        // Priest (marriage)
    Law,           // Law enforcement
    Skills,        // Skill trainer
    Quest          // Quest NPC
};
```

**Evidence:** `src/eodata.hpp:241-258`

### ENF Data Structure

```cpp
struct ENF_Data {
    int id;                // NPC ID (unique)
    std::string name;      // NPC name
    int graphic;           // Graphic ID
    
    short boss;            // Boss flag (HP multiplier)
    short child;           // Child NPC flag
    Type type;             // NPC type
    
    short vendor_id;       // Vendor/Quest ID
    
    // Combat stats
    int hp;                // Hit points
    int exp;               // Experience reward
    short mindam;          // Minimum damage
    short maxdam;          // Maximum damage
    short accuracy;        // Accuracy
    short evade;           // Evade
    short armor;           // Armor value
};
```

**Evidence:** `src/eodata.hpp:200-225`

### Arena Applications

#### Arena Master NPC
```cpp
ENF_Data arena_master;
arena_master.id = 500;
arena_master.name = "Arena Master";
arena_master.type = ENF::Quest;     // For quest dialog system
arena_master.vendor_id = 9000;      // Quest ID for arena menu
arena_master.graphic = 50;          // NPC sprite
arena_master.hp = 999999;           // Invulnerable
```

#### Arena Reward Vendor
```cpp
ENF_Data arena_vendor;
arena_vendor.id = 501;
arena_vendor.name = "Arena Rewards";
arena_vendor.type = ENF::Shop;
arena_vendor.vendor_id = 100;       // Shop ID
arena_vendor.graphic = 51;
```

#### Arena Spectator NPC
```cpp
ENF_Data arena_spectator;
arena_spectator.id = 502;
arena_spectator.name = "Arena Spectator";
arena_spectator.type = ENF::NPC;
arena_spectator.graphic = 52;
```

---

## 3. ESF - Spell File Format

**Purpose:** Defines all spells and skills  
**Location:** `data/pub/*.esf`  
**Record Size:** 51 bytes per spell

### Spell Types

```cpp
enum Type {
    Heal = 0,    // Healing spell
    Damage,      // Damage spell
    Bard         // Bard song/buff
};
```

### Target Restrictions

```cpp
enum TargetRestrict {
    NPCOnly = 0,   // Can only target NPCs
    Friendly,      // Can only target friendly players
    Opponent       // Can only target opponents
};
```

### Target Types

```cpp
enum Target {
    Normal = 0,  // Single target
    Self,        // Self-cast only
    Unknown1,
    Group        // Group/AoE spell
};
```

**Evidence:** `src/eodata.hpp:315-339`

### ESF Data Structure

```cpp
struct ESF_Data {
    int id;                        // Spell ID
    std::string name;              // Spell name
    std::string shout;             // Shout text (displayed when cast)
    
    short icon;                    // Icon graphic ID
    short graphic;                 // Cast effect graphic
    
    short tp;                      // TP cost
    short sp;                      // SP cost (skill points)
    
    unsigned char cast_time;       // Cast time (ticks)
    
    Type type;                     // Spell type
    TargetRestrict target_restrict; // Target restrictions
    Target target;                 // Target type
    
    short mindam;                  // Minimum damage/heal
    short maxdam;                  // Maximum damage/heal
    short accuracy;                // Accuracy modifier
    short hp;                      // HP cost (negative = heal)
};
```

**Evidence:** `src/eodata.hpp:285-313`

### Arena Applications

#### Arena Buff Spell
```cpp
ESF_Data arena_buff;
arena_buff.id = 100;
arena_buff.name = "Arena Vigor";
arena_buff.shout = "For glory!";
arena_buff.type = ESF::Heal;
arena_buff.target = ESF::Self;
arena_buff.target_restrict = ESF::Friendly;
arena_buff.tp = 50;
arena_buff.cast_time = 3;
arena_buff.hp = 100;  // Heal amount
```

#### Arena Attack Spell
```cpp
ESF_Data arena_blast;
arena_blast.id = 101;
arena_blast.name = "Arena Blast";
arena_blast.shout = "Take this!";
arena_blast.type = ESF::Damage;
arena_blast.target = ESF::Normal;
arena_blast.target_restrict = ESF::Opponent;
arena_blast.tp = 75;
arena_blast.cast_time = 5;
arena_blast.mindam = 80;
arena_blast.maxdam = 120;
arena_blast.accuracy = 90;
```

---

## 4. ECF - Class File Format

**Purpose:** Defines all character classes  
**Location:** `data/pub/*.ecf`  
**Record Size:** 14 bytes per class

### ECF Data Structure

```cpp
struct ECF_Data {
    int id;              // Class ID
    std::string name;    // Class name
    
    unsigned char base;  // Base stat value
    unsigned char type;  // Class type (?)
    
    // Stat bonuses per level
    short str;           // Strength gain
    short intl;          // Intelligence gain
    short wis;           // Wisdom gain
    short agi;           // Agility gain
    short con;           // Constitution gain
    short cha;           // Charisma gain
};
```

**Evidence:** `src/eodata.hpp:347-365`

### Arena Applications

Limited arena relevance - classes are character attributes. Could theoretically create "Arena Champion" class, but requires client-side class selection changes.

**Workaround:** Use titles instead of classes for arena progression.

---

## 5. EMF - Map File Format

**Purpose:** Defines individual map layout and properties  
**Location:** `data/maps/*.emf` (per map, e.g., `00001.emf`)  
**Format:** Complex binary structure

### Map Header

```
Offset | Size | Field
-------|------|------------------
0x00   | 3    | Magic ("EMF")
0x03   | 4    | RID
0x07   | 2    | Length
0x09   | 1    | Version
...
0x1F   | 1    | PK flag (3 = PK enabled)
0x20   | 1    | Effect type
...
0x25   | 1    | Width - 1
0x26   | 1    | Height - 1
...
0x2A   | 1    | Scroll (music/ambiance)
0x2B   | 1    | Relog X (spawn point)
0x2C   | 1    | Relog Y (spawn point)
```

**Evidence:** `src/map.cpp:469-523`

### Map Data Sections

Maps contain multiple data sections in order:

1. **NPC Spawns** - NPC spawn locations and parameters
2. **Unknown Section** - Skipped
3. **Unknown Section** - Skipped
4. **Tile Specs** - Special tile properties
5. **Warps** - Warp tile destinations
6. **Unknown Section** - Skipped (copy of NPC spawns?)
7. **Chest Spawns** - Chest item spawns

**Evidence:** `src/map.cpp:523-721`

### Map Tile Specs

```cpp
enum TileSpec {
    None = -1,
    Wall,              // Unwalkable
    ChairDown,         // Chair facing down
    ChairLeft,
    ChairRight,
    ChairUp,
    ChairDownRight,
    ChairUpLeft,
    ChairAll,
    Door,              // Door tile
    Chest,             // Chest tile
    Unknown1-6,
    BankVault,         // Bank access
    NPCBoundary,       // NPC movement boundary
    MapEdge,           // Map edge
    FakeWall,          // Appears as wall but walkable
    Board1-8,          // Message board tiles (8 types)
    Jukebox,           // Music jukebox
    Jump,              // Jump tile
    Water,             // Water tile
    Unknown7,
    Arena,             // Arena tile (spawn trigger)
    AmbientSource,     // Ambient sound source
    Spikes1-3          // Spike trap tiles
};
```

**Evidence:** `src/map.hpp:76-115`

### Map Properties

```cpp
struct Map {
    int id;                  // Map ID
    unsigned char width;     // Map width
    unsigned char height;    // Map height
    
    bool pk;                 // PK enabled
    EffectType effect;       // Visual effect
    
    unsigned char scroll;    // Music/ambiance ID
    unsigned char relog_x;   // Respawn X
    unsigned char relog_y;   // Respawn Y
    
    std::vector<Map_Tile> tiles;    // Tile data
    std::list<NPC*> npcs;           // Active NPCs
    std::list<Map_Item> items;      // Items on ground
    std::list<Map_Chest> chests;    // Chests
    std::list<Character*> characters; // Players on map
};
```

**Evidence:** `src/map.hpp:211-292`

### Arena Applications

#### Arena Map Requirements
- **PK Disabled:** Arena maps should have PK=false (arena system handles combat)
- **Relog Point:** Safe spawn point for eliminated players
- **Arena Tiles:** Spawn trigger tiles
- **Board Tiles:** Leaderboard access points
- **Boundaries:** Use Wall or MapEdge to contain arena

#### Arena Map Layout Example
```
Lobby Area:
  - Board1 tile at (10, 5) - Leaderboard
  - Arena tile at (10, 10) - Spawn point 1
  - Arena tile at (10, 15) - Spawn point 2
  - Door tile at (5, 10) - Exit to main world

Combat Area:
  - Open space (20x20 grid)
  - Wall tiles as boundaries
  - No special tiles (pure combat)
```

---

## File Modification Tools

### Official Tools
- **EO Item Editor** - Edit EIF files
- **EO NPC Editor** - Edit ENF files  
- **EO Spell Editor** - Edit ESF files
- **EO Class Editor** - Edit ECF files
- **EO Map Editor** - Edit EMF files

### Programmatic Editing
ArenaServ can read all formats. For writing:
- Modify structures in memory
- Call `.Write()` methods (if implemented)
- Or use external editors

**Evidence:** `src/eodata.cpp` contains Read() functions

---

## File Sync & Caching

### Client-Side Caching
Clients cache pub files in:
- `data/pub/*.eif/enf/esf/ecf`
- Maps: temporary per-warp

### Invalidation
To force client redownload:
1. **Change RID** - Edit 4-byte RID value at offset 0x03
2. **Restart server** - Server sends new RID
3. **Client mismatch** - Client detects different RID, downloads new file

### Testing Changes
```bash
# After editing EIF/ENF/ESF/ECF:
1. Change RID (increment any byte)
2. Restart server
3. Delete client cache (optional, auto-redownloads)
4. Connect to test
```

---

## Data File Constraints

### Server-Side Editable ✅
- **EIF** - Can add items freely (ID 1-65535)
- **ENF** - Can add NPCs freely (ID 1-65535)
- **ESF** - Can add spells freely (ID 1-65535)
- **ECF** - Can add classes (ID 1-65535)

### Client Behavior Constraints ⚠️
- **Graphics** - Must reference existing GFX files
- **Item Types** - Client interprets types (can't invent new types)
- **NPC Types** - Client handles specific types (Shop, Quest, etc.)
- **Spell Effects** - Client renders predefined effect IDs

### Cannot Change Without Client Mod ❌
- **New item types** - E.g., "Arena Badge" type
- **New NPC behaviors** - E.g., "Spectator" AI
- **New spell mechanics** - E.g., "Arena-only" spell
- **New tile specs** - E.g., "Spectator Zone" tile

### Workarounds
- **Reuse existing types** - Arena Token = Static item with Lore flag
- **Type overloading** - Quest NPC for arena menus
- **Server-side logic** - Arena rules in code, not data files

---

## Arena Data File Strategy

### Recommended Approach

#### 1. Items (EIF)
- **ID Range:** 5000-5999 (arena items)
- **Token:** ID 5000, Static/Money type
- **Trophies:** ID 5001-5020, Static/Unique
- **Rewards:** ID 5021-5100, equipment with arena theme

#### 2. NPCs (ENF)
- **ID Range:** 500-599 (arena NPCs)
- **Arena Master:** ID 500, Quest type, vendor_id 9000
- **Reward Vendor:** ID 501, Shop type
- **Stat Checker:** ID 502, Quest type, vendor_id 9001
- **Queue Manager:** ID 503, Quest type, vendor_id 9002

#### 3. Spells (ESF)
- **ID Range:** 100-199 (arena spells, if needed)
- **Arena-specific buffs/abilities**
- **Or skip entirely for melee-only arena**

#### 4. Maps (EMF)
- **Arena Lobby:** Map 100
- **Arena 1v1:** Map 101
- **Arena FFA:** Map 102
- **Arena Team:** Map 103

---

## References

### Source Files
- `src/eodata.hpp` - Data structures
- `src/eodata.cpp` - File reading
- `src/map.cpp` - Map loading
- `src/map.hpp` - Map structures

### Data Locations
- `data/pub/` - Example pub files
- `data/maps/` - Map files
- `config/files.ini` - File path configuration

### External Resources
- **EO Protocol Spec:** `references/eo-protocol-master/`
- **Client Reference:** `references/Client/Endless.exe`

---

**Summary:** EO pub files (EIF/ENF/ESF/ECF/EMF) are binary formats defining items, NPCs, spells, classes, and maps. Server can freely create custom definitions that work with vanilla client, with constraints on behavior types (must reuse existing types). Arena features should use ID ranges 5000+ for items, 500+ for NPCs, dedicated maps 100-199. All data synced server→client automatically.
