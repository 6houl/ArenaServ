# Client Capability Matrix

**Status:** CONFIRMED (based on ArenaServ source code analysis)  
**Last Updated:** 2026-09-22  
**Client:** Vanilla Endless Online (unmodified)

## Purpose

This document catalogs **every confirmed capability** of the vanilla EO client that can be triggered by the server. This is the authoritative reference for determining if a feature can be implemented server-side.

## Evidence Classification

- ✅ **CONFIRMED** - Implemented in ArenaServ, packet exists, tested
- 🔶 **PROBABLE** - Evidence in code, not yet tested
- ❓ **EXPERIMENTAL** - Needs validation
- ❌ **CLIENT-REQUIRED** - Confirmed impossible without client changes
- ⚠️ **UNKNOWN** - Insufficient evidence

---

## Display & UI Capabilities

### Text Display Systems

#### 1. Status Message Popup
**Status:** ✅ CONFIRMED  
**Packet:** `PACKET_MESSAGE (35), PACKET_OPEN`  
**Server Function:** `Character::StatusMsg(string)`  
**Client Surface:** Yellow popup message box (bottom of screen)  
**Capabilities:**
- Single line text
- Automatic word wrap
- Length limit: `ChatMaxWidth` config
- Auto-dismisses after timeout
- Non-blocking

**Evidence:**
- File: `src/character.cpp:600-608`
- Implementation:
```cpp
void Character::StatusMsg(std::string message) {
    message = util::text_cap(message, static_cast<int>(this->world->config["ChatMaxWidth"]));
    PacketBuilder builder(PACKET_MESSAGE, PACKET_OPEN, message.length());
    builder.AddString(message);
    this->Send(builder);
}
```

**ArenaServ Use Cases:**
- Arena cooldown notifications
- Capacity warnings
- Match status updates
- Error messages

---

#### 2. Server Chat Message
**Status:** ✅ CONFIRMED  
**Packet:** `PACKET_TALK (18), PACKET_SERVER (23)`  
**Server Function:** `Character::ServerMsg(string)`, `World::ServerMsg(string)`  
**Client Surface:** Chat window with "Server" prefix (yellow text)  
**Capabilities:**
- Appears in chat log
- Persistent in scrollback
- Length limit: `ChatMaxWidth` - "Server  " width
- Can be sent to individual or broadcast
- Scrollable history

**Evidence:**
- File: `src/character.cpp:591-599`
- File: `src/world.cpp:1248-1258`

**ArenaServ Use Cases:**
- Match results announcements
- Kill notifications
- Leaderboard updates
- Global arena events

---

#### 3. Info Box / Dialog
**Status:** ✅ CONFIRMED  
**Packet:** `PACKET_MESSAGE (35), PACKET_ACCEPT (2)`  
**Server Function:** `Character::ShowInfoBox(title, content)` or `ShowInfoBox(title, lines)`  
**Client Surface:** Scrollable dialog window with title bar  
**Capabilities:**
- Title bar (string)
- Multi-line content (unlimited length)
- Automatic word wrapping (197 pixel width)
- Scrollable vertically
- Modal dialog (blocks other actions)
- Close button

**Evidence:**
- File: `src/character.cpp:2068-2189`
- Implementation includes sophisticated word-wrap algorithm
- Used by `$audcmd` admin command

**Format:**
```cpp
// Simple version
ShowInfoBox("Title", "Content with\nmanual\nline breaks");

// Advanced version with automatic formatting
ShowInfoBox("Title", {
    "Line 1",
    "Line 2 with automatic wrapping if too long",
    "Line 3"
});
```

**ArenaServ Use Cases:**
- Match summaries
- Detailed statistics
- Arena rules/help
- Leaderboards (formatted text)
- Match history logs

---

#### 4. Map Local Chat
**Status:** ✅ CONFIRMED  
**Packet:** `PACKET_TALK (18), PACKET_MSG (15)`  
**Server Function:** `Map::Msg(Character* from, string)`  
**Client Surface:** Chat window with character name prefix  
**Capabilities:**
- Shows sender name
- Only visible to players on same map
- Length limit: `ChatLength` config
- Chat bubble over character (optional)
- Appears in chat log

**Evidence:**
- File: `src/map.cpp` (Msg function)
- Used for local communication

**ArenaServ Use Cases:**
- Arena-local announcements
- In-match commentary
- Spectator chat (if implemented)

---

#### 5. Admin Chat
**Status:** ✅ CONFIRMED  
**Packet:** `PACKET_TALK (18), PACKET_ADMIN (17)`  
**Server Function:** `World::AdminMsg(from, message, minlevel)`  
**Client Surface:** Admin channel in chat (requires admin level)  
**Capabilities:**
- Filtered by admin level
- Only visible to admins >= minlevel
- Different color than normal chat
- Persistent log

**Evidence:**
- File: `src/world.cpp:1202-1246`
- Used for admin reports and requests

**ArenaServ Use Cases:**
- Admin notifications of arena issues
- Match monitoring logs
- Anti-cheat alerts

---

### Window-Based UI Systems

#### 6. Message Board System
**Status:** ✅ CONFIRMED  
**Packet:** `PACKET_BOARD (43), PACKET_OPEN (13)`  
**Server Function:** `Character::ShowBoard(Board*)`  
**Client Surface:** Message board window  
**Capabilities:**
- List of "posts" (subjects)
- Each post: ID, Author, Subject line, Timestamp
- Click to read full body
- Subject length limit: `BoardMaxSubjectLength`
- Body length limit: `BoardMaxPostLength`
- Board ID (1-8, tiles Board1-Board8)
- Can create/delete posts (configurable)

**Evidence:**
- File: `src/character.cpp:1402-1459`
- File: `src/handlers/Board.cpp`
- Post structure in `src/world.hpp:71-81`

**Packet Structure:**
```cpp
PacketBuilder builder(PACKET_BOARD, PACKET_OPEN, size);
builder.AddChar(board_id + 1);
builder.AddChar(num_posts);
// For each post:
builder.AddShort(post_id);
builder.AddByte(255);
builder.AddBreakString(author);
builder.AddBreakString(subject);
```

**Reading Post Body:**
```cpp
PacketBuilder reply(PACKET_BOARD, PACKET_PLAYER, 2 + body.length());
reply.AddShort(post_id);
reply.AddString(body);
```

**ArenaServ Use Cases:**
- **Leaderboards** - Each "post" is a rank entry
- **Match History** - Posts are past matches
- **Tournament Brackets** - Visual bracket via formatted posts
- **Arena News** - Updates and announcements

**Limitations:**
- Requires board tile on map OR virtual board (untested)
- Fixed window layout
- Limited sorting options

---

#### 7. Quest Dialog System
**Status:** ✅ CONFIRMED  
**Packet:** `PACKET_QUEST (50), PACKET_DIALOG (34)`  
**Server Function:** Custom packet building  
**Client Surface:** Quest dialog window with NPC portrait  
**Capabilities:**
- NPC vendor ID (controls portrait)
- Multiple dialog pages
- Dialog text (unlimited length)
- Quest list view
- Progress tracking
- Quest links/navigation

**Evidence:**
- File: `src/handlers/Quest.cpp:60-119`
- EO+ quest system implementation

**Packet Structure:**
```cpp
PacketBuilder builder(PACKET_QUEST, PACKET_DIALOG, size);
builder.AddChar(dialog_count);
builder.AddShort(vendor_id);  // NPC for portrait
builder.AddByte(255);
builder.AddBreakString("NPC Name");
builder.AddBreakString("Dialog text");
// Repeat for each dialog page
```

**Quest List:**
```cpp
PacketBuilder reply(PACKET_QUEST, PACKET_LIST, size);
reply.AddChar(page);  // In Progress, History, etc.
reply.AddShort(quest_count);
// For each quest:
reply.AddShort(quest_id);
reply.AddBreakString("Quest name/description");
```

**ArenaServ Use Cases:**
- **Arena Menus** - Multi-option dialog (View Stats, Enter Queue, etc.)
- **Interactive NPCs** - Arena Master dialogs
- **Quest-like progression** - Arena achievements as quests
- **Match briefings** - Pre-match information

---

#### 8. Shop Interface
**Status:** ✅ CONFIRMED  
**Packet:** `PACKET_SHOP (13), PACKET_OPEN (13)`  
**Server Function:** Custom packet building (NPC shops)  
**Client Surface:** Shop window with buy/sell/craft tabs  
**Capabilities:**
- Shop name
- Player "gold" display (can repurpose)
- Item list with icons (from EIF)
- Item prices (can repurpose as points/scores)
- Buy/Sell/Craft tabs
- Item descriptions from EIF

**Evidence:**
- File: `src/handlers/Shop.cpp:173-215`
- Data: `data/shops.ini`

**Packet Structure:**
```cpp
PacketBuilder builder(PACKET_SHOP, PACKET_OPEN, size);
builder.AddShort(shop_npc_id);
builder.AddThree(player_gold);  // Or "arena points"
builder.AddChar(shop_id);
builder.AddBreakString("Shop Name");
// Trade items
builder.AddShort(trade_item_count);
for (each trade_item) {
    builder.AddShort(item_id);  // From EIF
    builder.AddInt(buy_price);
    builder.AddInt(sell_price);
    builder.AddChar(max_buy);
}
// Craft items
builder.AddShort(craft_item_count);
for (each craft_item) {
    builder.AddShort(result_item_id);
    builder.AddChar(num_ingredients);
    for (each ingredient) {
        builder.AddShort(ingredient_item_id);
        builder.AddChar(ingredient_amount);
    }
}
```

**ArenaServ Use Cases:**
- **Reward Shop** - Arena points to buy items/rewards
- **Trophy Vendor** - Purchase cosmetic trophies
- **Stat Display** - "Prices" show ELO/rank (creative reuse)
- **Crafting = Ranking** - Craft recipes show progression paths

**Limitations:**
- Requires items in EIF file
- Fixed shop window layout
- "Currency" shown as gold icon

---

#### 9. Book / Character Info
**Status:** ✅ CONFIRMED  
**Packet:** `PACKET_BOOK (51), PACKET_REPLY (3)`  
**Server Function:** Used by `$charinfo` command  
**Client Surface:** Character information window  
**Capabilities:**
- Character name
- Home (string field)
- Partner (string field)
- Title (string field)
- Guild (string field)
- Guild rank (string field)
- Class icon
- Gender icon
- Admin level icon (optional)

**Evidence:**
- File: `src/handlers/Book.cpp:36-105`
- File: `src/commands/info.cpp:166-235`

**Packet Structure:**
```cpp
PacketBuilder builder(PACKET_BOOK, PACKET_REPLY, size);
builder.AddBreakString(name);
builder.AddBreakString(home);
builder.AddBreakString(partner);
builder.AddBreakString(title);
builder.AddBreakString(guild_name);
builder.AddBreakString(guild_rank);
builder.AddChar(class_id);
builder.AddChar(gender);
builder.AddChar(admin_icon);  // 0 = none, ICON_* constants
```

**ArenaServ Use Cases:**
- **Arena Profile** - Repurpose fields for arena stats
  - Home → "Arena Rank #42"
  - Title → "Arena Champion"
  - Guild → "1850 ELO"
  - Guild Rank → "Win Rate: 65%"
- Limited but possible

**Limitations:**
- Fixed fields, can't add new ones
- Limited to string replacements
- Class/gender icons not changeable

---

#### 10. Paperdoll / Equipment Display
**Status:** ✅ CONFIRMED  
**Packet:** `PACKET_PAPERDOLL (30), PACKET_REPLY (3)`  
**Server Function:** Character equipment display  
**Client Surface:** Equipment paperdoll window  
**Capabilities:**
- Character name
- Home string
- Partner string
- Title string
- Guild tag
- Class icon
- Gender/appearance
- 15 equipment slots (item graphics from EIF)

**Evidence:**
- File: `src/handlers/Paperdoll.cpp`
- Equipment slots defined in `src/character.hpp:313-327`

**ArenaServ Use Cases:**
- **Trophy Display** - Equipment slots show arena achievements
- **Badge System** - Equip "badges" for accomplishments
- Requires trophy/badge items in EIF

**Limitations:**
- Requires items in EIF
- Fixed to 15 equipment slots
- Shows character avatar

---

### Interaction Systems

#### 11. NPC Dialog Interaction
**Status:** ✅ CONFIRMED  
**Packet:** `PACKET_NPC (26)` various actions  
**Server Function:** Quest system, shops, banks, etc.  
**Client Surface:** Various dialogs triggered by NPC interaction  
**Capabilities:**
- Talk to NPC (opens quest dialog)
- Shop interaction
- Bank interaction
- Inn interaction
- Skill master
- Barber
- Law enforcement
- Priest (marriage)

**Evidence:**
- File: `src/handlers/Quest.cpp`, `src/quest.cpp`
- File: `src/handlers/Shop.cpp`
- File: various NPC type handlers

**ArenaServ Use Cases:**
- **Arena Master NPC** - Central arena interface
- **Stat Checker NPC** - View your stats
- **Reward Vendor NPC** - Redeem points
- **Queue Manager NPC** - Enter/leave queue

---

#### 12. Map Tile Interactions
**Status:** ✅ CONFIRMED  
**Packet:** Various based on tile type  
**Server Function:** `Map::GetTile(x,y).tilespec`  
**Client Surface:** Automatic on walk/click  
**Capabilities:**
- **Board tiles** (Board1-Board8) - Open message board
- **Chest tiles** - Open chest
- **BankVault** - Open bank
- **Jukebox** - Play music
- **Door** - Open/close door
- **Chair** - Sit down
- **Arena** - Special tile (behavior in code)
- **Spikes** - Damage over time
- **Jump** - Teleport
- **Warp** - Map transition

**Evidence:**
- File: `src/map.hpp:73-112` (TileSpec enum)
- File: `src/map.cpp` (tile behavior)

**ArenaServ Use Cases:**
- **Arena tiles** - Spawn points, boundaries
- **Board tiles** - Leaderboard displays
- **Custom interactions** - Trigger events on step

---

### Combat & Action Systems

#### 13. Attack Animation
**Status:** ✅ CONFIRMED  
**Packet:** `PACKET_ATTACK (11), PACKET_PLAYER (8)`  
**Server Function:** `Map::Attack(Character*, Direction)`  
**Client Surface:** Character attack animation  
**Capabilities:**
- Direction-based
- Weapon-dependent animation
- Sound effects
- Hit detection (server-side)

**Evidence:**
- File: `src/map.cpp:1487-1650` (Attack function)

---

#### 14. Spell/Skill Animation
**Status:** ✅ CONFIRMED  
**Packet:** `PACKET_SPELL (12)` / `PACKET_CAST (44)`  
**Server Function:** `Map::SpellSelf/SpellAttack/SpellGroup`  
**Client Surface:** Spell animation and effects  
**Capabilities:**
- Target selection (self, other, group, NPC)
- Visual effects from ESF
- TP consumption
- Cast time
- Cooldown

**Evidence:**
- File: `src/map.cpp` (Spell functions)
- File: `src/handlers/Spell.cpp`

---

#### 15. Warp Animation
**Status:** ✅ CONFIRMED  
**Packet:** `PACKET_WARP (19), PACKET_REQUEST (1)`  
**Server Function:** `Character::Warp(map, x, y, animation)`  
**Client Surface:** Warp visual effect  
**Capabilities:**
- WARP_ANIMATION_NONE
- WARP_ANIMATION_SCROLL (default map entry)
- WARP_ANIMATION_ADMIN (bubbles, optional with config)

**Evidence:**
- File: `src/eodata.hpp:484-489` (WarpAnimation enum)
- File: `src/character.cpp:1286-1380` (Warp function)

**ArenaServ Use Cases:**
- **Arena teleportation** - Enter/exit effects
- **Match start** - Dramatic entry
- **Elimination** - Forced warp out

---

### Visual Feedback Systems

#### 16. Effect Display
**Status:** ✅ CONFIRMED  
**Packet:** `PACKET_EFFECT (31), PACKET_ADMIN/PLAYER`  
**Server Function:** `Character::Effect(int effect, bool echo)`  
**Client Surface:** Visual effect on character  
**Capabilities:**
- Effect ID from ESF or predefined
- Level-up effect
- Spell effects
- Damage indicators
- Heal indicators

**Evidence:**
- File: `src/character.cpp:813-830`

---

#### 17. Emote System
**Status:** ✅ CONFIRMED  
**Packet:** `PACKET_EMOTE (9)`  
**Server Function:** `Character::Emote(enum Emote, bool echo)`  
**Client Surface:** Character emote animation  
**Capabilities:**
- Predefined emotes (happy, sad, etc.)
- Visual animation above character
- Can be broadcast to nearby players

**Evidence:**
- File: `src/character.cpp:799-811`

**ArenaServ Use Cases:**
- Victory celebrations
- Match start rituals

---

#### 18. Sitting/Standing
**Status:** ✅ CONFIRMED  
**Packet:** `PACKET_SIT (41), PACKET_CHAIR (8)`  
**Server Function:** `Character::Sit(SitState)`, `Character::Stand()`  
**Client Surface:** Character sitting animation  
**Capabilities:**
- SIT_STAND (standing)
- SIT_FLOOR (sitting on ground)
- SIT_CHAIR (sitting in chair)
- Different chair types from tiles

**Evidence:**
- File: `src/character.cpp:785-797`
- File: `src/map.hpp:76-82` (Chair tile specs)

---

## Data-Driven Capabilities

### 19. Custom Items (EIF)
**Status:** ✅ CONFIRMED  
**Format:** EIF (Endless Item File)  
**Server:** Broadcasts EIF to clients on connect  
**Client:** Reads item graphics, stats, descriptions  
**Capabilities:**
- Create new items (ID, name, type, graphics)
- Item stats (HP, TP, damage, armor, etc.)
- Item requirements (level, class, stats)
- Special effects
- Visual appearance (dollgraphic)

**Evidence:**
- File: `src/eodata.hpp:17-153` (EIF_Data structure)
- File: `data/pub/empty.eif`

**ArenaServ Use Cases:**
- **Arena Tokens** - Currency items
- **Trophy Items** - Achievement badges
- **Reward Items** - Prizes for winning
- **Cosmetic Items** - Arena-exclusive gear

**Process:**
1. Edit EIF with item editor
2. Add item definitions
3. Server broadcasts updated EIF
4. Client displays new items

---

### 20. Custom NPCs (ENF)
**Status:** ✅ CONFIRMED  
**Format:** ENF (Endless NPC File)  
**Server:** Broadcasts ENF to clients  
**Client:** Reads NPC graphics, behavior types  
**Capabilities:**
- Create new NPCs (ID, name, graphics)
- NPC types (passive, aggressive, shop, bank, etc.)
- NPC stats (HP, damage, etc.)
- Shop associations
- Quest associations

**Evidence:**
- File: `src/eodata.hpp:248-380` (ENF structures)
- File: `data/pub/empty.enf`

**ArenaServ Use Cases:**
- **Arena Master NPC** - Central arena NPC
- **Reward Vendor NPC** - Sells rewards
- **Stat Checker NPC** - Displays statistics
- **Queue Manager NPC** - Handles match queuing

---

### 21. Custom Spells (ESF)
**Status:** ✅ CONFIRMED  
**Format:** ESF (Endless Spell File)  
**Server:** Broadcasts ESF to clients  
**Client:** Reads spell graphics, effects  
**Capabilities:**
- Create new spells
- Spell animations
- Target types (self, ally, enemy, group)
- Stat requirements
- TP cost
- Cast time

**Evidence:**
- File: `src/eodata.hpp:382-475` (ESF structures)
- File: `data/pub/empty.esf`

**ArenaServ Use Cases:**
- **Arena Buffs** - Temporary power-ups
- **Arena-specific abilities** - Special moves
- Limited use in pure melee arena

---

### 22. Custom Maps (EMF)
**Status:** ✅ CONFIRMED  
**Format:** EMF (Endless Map File)  
**Server:** Loads maps from `data/maps/*.emf`  
**Client:** Receives map on warp  
**Capabilities:**
- Tile types (walkable, wall, special)
- Warp tiles (to other maps)
- Item spawns
- NPC spawns
- Chest spawns
- Tile specs (board, chair, arena, etc.)

**Evidence:**
- File: `src/map.cpp:357-729` (Map::Load)
- File: `data/maps/` directory

**ArenaServ Use Cases:**
- **Arena Maps** - Custom battle arenas
- **Spawn Tiles** - Entry points
- **Arena Tiles** - Special mechanics
- **Warp Tiles** - Quick exits
- **Board Tiles** - Leaderboard locations

---

## NOT Possible Without Client Modification

### ❌ New UI Windows
- Cannot create new window types
- Must use existing packet-triggered windows

### ❌ New Packet Types
- Client only handles predefined packets
- Cannot add PACKET_CUSTOM_ARENA

### ❌ Custom Graphics Rendering
- Cannot add new sprite rendering
- Cannot modify existing rendering pipeline

### ❌ New Input Handling
- Cannot add new keyboard shortcuts
- Cannot add new mouse interactions
- Limited to existing controls

### ❌ Real-Time HUD Elements
- Cannot add health bars above enemies
- Cannot add arena timer on screen
- Cannot add live match score overlay

### ❌ Custom Camera Controls
- Cannot add spectator camera
- Cannot add replay controls
- Cannot modify view distance

### ❌ Audio Beyond Existing
- Cannot add new sound formats
- Cannot trigger sounds beyond existing SFX IDs
- Cannot play custom music files

---

## Summary Table

| Capability | Status | Packet | Reusable for Arena |
|-----------|--------|--------|-------------------|
| Status Message | ✅ | MESSAGE/OPEN | Yes - notifications |
| Server Chat | ✅ | TALK/SERVER | Yes - announcements |
| Info Box | ✅ | MESSAGE/ACCEPT | Yes - stats, logs |
| Map Chat | ✅ | TALK/MSG | Yes - local events |
| Message Board | ✅ | BOARD/OPEN | Yes - leaderboards |
| Quest Dialog | ✅ | QUEST/DIALOG | Yes - menus |
| Shop | ✅ | SHOP/OPEN | Yes - rewards |
| Book | ✅ | BOOK/REPLY | Limited - profiles |
| Paperdoll | ✅ | PAPERDOLL | Limited - trophies |
| Custom Items | ✅ | Data file | Yes - tokens/rewards |
| Custom NPCs | ✅ | Data file | Yes - arena NPCs |
| Custom Maps | ✅ | Data file | Yes - arenas |

---

**Key Insight:** The vanilla client has MORE display capabilities than initially apparent. By creatively using message boards, quest dialogs, shops, and info boxes, extensive arena features can be implemented server-side.
