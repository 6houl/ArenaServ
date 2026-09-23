# EO Protocol Packet Catalog

**Status:** CONFIRMED (based on ArenaServ source code)  
**Last Updated:** 2026-09-22  
**Purpose:** Complete reference of all packet families and actions with arena relevance

## Overview

The Endless Online protocol uses a two-byte addressing system:
- **Packet Family** (byte 1): Major category (e.g., PACKET_WALK, PACKET_ATTACK)
- **Packet Action** (byte 2): Specific operation (e.g., PACKET_REQUEST, PACKET_REPLY)

**Evidence:** `src/fwd/packet.hpp`

---

## Packet Families (0-51, 255)

### Core Families

| ID | Name | Purpose | Arena Relevance |
|----|------|---------|----------------|
| 0 | INTERNAL | Internal server messages | ⭐⭐ Queue/state management |
| 1 | CONNECTION | Connection handshake | ❌ Not arena-specific |
| 2 | ACCOUNT | Account operations | ❌ Not arena-specific |
| 3 | CHARACTER | Character selection/creation | ❌ Not arena-specific |
| 4 | LOGIN | Login process | ❌ Not arena-specific |
| 5 | WELCOME | World entry | ⭐ Spawn location |
| 6 | WALK | Movement | ⭐⭐⭐⭐⭐ Core arena mechanic |
| 7 | FACE | Change direction | ⭐⭐⭐ Arena positioning |
| 8 | CHAIR | Sitting | ⭐ Spectator mode |
| 9 | EMOTE | Emotes | ⭐⭐ Victory celebrations |
| 11 | ATTACK | Physical attacks | ⭐⭐⭐⭐⭐ Core arena combat |
| 12 | SPELL | Spell casting (self-cast) | ⭐⭐⭐⭐ Arena combat |
| 13 | SHOP | Shop interface | ⭐⭐⭐ Reward vendor |
| 14 | ITEM | Item usage/drops | ⭐⭐⭐⭐ Arena items/potions |
| 16 | STATSKILL | Stats/skills | ⭐⭐ Arena buffs/abilities |
| 17 | GLOBAL | Global messages | ⭐⭐ Announcements |
| 18 | TALK | Chat messages | ⭐⭐⭐⭐ Arena chat |
| 19 | WARP | Map transitions | ⭐⭐⭐⭐⭐ Arena entry/exit |
| 21 | JUKEBOX | Music control | ⭐ Arena ambiance |
| 22 | PLAYERS | Player list | ⭐⭐ Arena roster |
| 23 | AVATAR | Player appearance | ❌ Cosmetic only |
| 24 | PARTY | Party system | ⭐⭐⭐ Team arenas |
| 25 | REFRESH | Entity updates | ⭐⭐⭐ Arena state sync |
| 26 | NPC | NPC interaction | ⭐⭐⭐ Arena Master |
| 27 | PLAYER_RANGE | Player enters/leaves range | ⭐⭐⭐ Arena visibility |
| 28 | NPC_RANGE | NPC enters/leaves range | ❌ Rare in arenas |
| 29 | RANGE | Combined range updates | ⭐⭐⭐ Arena visibility |
| 30 | PAPERDOLL | Equipment display | ⭐⭐ Trophy display |
| 31 | EFFECT | Visual effects | ⭐⭐⭐ Combat feedback |
| 32 | TRADE | Player trading | ❌ Disabled in arena |
| 33 | CHEST | Chest interaction | ⭐ Arena rewards |
| 34 | DOOR | Door interaction | ⭐ Arena gates |
| 35 | MESSAGE | Status messages/dialogs | ⭐⭐⭐⭐⭐ Arena UI |
| 36 | BANK | Bank interaction | ❌ Not arena-specific |
| 37 | LOCKER | Locker interaction | ❌ Not arena-specific |
| 38 | BARBER | Appearance change | ❌ Not arena-specific |
| 39 | GUILD | Guild operations | ⭐ Team affiliation |
| 40 | MUSIC | Music control | ⭐ Arena music |
| 41 | SIT | Sitting action | ⭐ Spectator mode |
| 42 | RECOVER | HP/TP recovery | ⭐⭐⭐⭐ Arena respawn |
| 43 | BOARD | Message boards | ⭐⭐⭐⭐⭐ Leaderboards |
| 44 | CAST | Targeted spell casting | ⭐⭐⭐⭐⭐ Arena combat |
| 45 | ARENA | **Arena-specific packets** | ⭐⭐⭐⭐⭐ Legacy system |
| 46 | PRIEST | Marriage/divorce | ❌ Not arena-specific |
| 47 | MARRIAGE | Marriage operations | ❌ Not arena-specific |
| 48 | ADMININTERACT | Admin actions | ⭐⭐ Arena moderation |
| 49 | CITIZEN | Citizenship operations | ❌ Not arena-specific |
| 50 | QUEST | Quest system | ⭐⭐⭐⭐⭐ Arena menus |
| 51 | BOOK | Character info | ⭐⭐ Arena profiles |
| 255 | F_INIT | File initialization | ❌ Login only |

---

## Packet Actions (1-34, 128-129, 240-242, 255)

### Standard Actions

| ID | Name | Typical Use |
|----|------|------------|
| 1 | REQUEST | Client requests action |
| 2 | ACCEPT | Server accepts/confirms |
| 3 | REPLY | Server replies with data |
| 4 | REMOVE | Remove entity/data |
| 5 | AGREE | Confirm agreement |
| 6 | CREATE | Create new entity |
| 7 | ADD | Add entity/data |
| 8 | PLAYER | Player-specific data |
| 9 | TAKE | Take/retrieve item |
| 10 | USE | Use item/feature |
| 11 | BUY | Purchase action |
| 12 | SELL | Sell action |
| 13 | OPEN | Open interface |
| 14 | CLOSE | Close interface |
| 15 | MSG | Message/chat |
| 16 | SPEC | Special action |
| 17 | ADMIN | Admin message |
| 18 | LIST | List data |
| 20 | TELL | Private message |
| 21 | REPORT | Report to admins |
| 22 | ANNOUNCE | Announcement |
| 23 | SERVER | Server message |
| 24 | DROP | Drop item |
| 25 | JUNK | Destroy item |
| 26 | OBTAIN | Obtain item |
| 27 | GET | Get item |
| 28 | KICK | Kick player |
| 29 | RANK | Rank/rating data |
| 30 | TARGET_SELF | Target self |
| 31 | TARGET_OTHER | Target other entity |
| 33 | TARGET_GROUP | Target group |
| 34 | DIALOG | Dialog interaction |

### Internal Actions

| ID | Name | Purpose |
|----|------|---------|
| 128 | INTERNAL_NULL | Null internal message |
| 129 | INTERNAL_WARP | Internal warp trigger |

### Network Actions

| ID | Name | Purpose |
|----|------|---------|
| 240 | PING | Connection keepalive |
| 241 | PONG | Ping response |
| 242 | NET3 | Network protocol 3 |
| 255 | A_INIT | Action initialization |

---

## Arena-Critical Packets

### 1. PACKET_ARENA (45)

**Legacy Arena System**

#### ARENA, REQUEST (45, 1)
**Direction:** Client → Server  
**Purpose:** Request to enter arena

**Expected Data:**
```cpp
// Client sends arena entry request
// Exact format depends on implementation
```

**Evidence:** `src/fwd/packet.hpp:59` (packet family exists)

#### ARENA, ACCEPT (45, 2)
**Direction:** Server → Client  
**Purpose:** Confirm arena entry

**Status:** ⚠️ DEPRECATED - Legacy system, limited documentation in current codebase

**Arena Class Reference:** `src/arena.cpp`, `src/arena.hpp`

---

### 2. PACKET_WALK (6) - Movement

#### WALK, PLAYER (6, 8)
**Direction:** Server → Client  
**Purpose:** Notify player movement

**Server Code:**
```cpp
PacketBuilder builder(PACKET_WALK, PACKET_PLAYER, 5);
builder.AddShort(player_id);
builder.AddChar(direction);  // 0=down, 1=left, 2=up, 3=right
builder.AddChar(x);
builder.AddChar(y);
```

**Arena Relevance:** ⭐⭐⭐⭐⭐
- Core movement tracking
- Position validation
- Collision detection
- Bounds enforcement

**Evidence:** `src/handlers/Walk.cpp`

#### WALK, REQUEST (6, 1)
**Direction:** Client → Server  
**Purpose:** Request to move

**Client Data:**
```cpp
byte walk_action;  // 0=walk, 1=run (?)
byte direction;
```

**Server Validation:**
- Check tile walkability
- Check map bounds
- Check speed/cooldown
- Apply arena boundaries

---

### 3. PACKET_ATTACK (11) - Physical Combat

#### ATTACK, USE (11, 10)
**Direction:** Client → Server  
**Purpose:** Initiate attack

**Client Data:**
```cpp
byte direction;  // Attack direction
byte timestamp;  // For anti-spam
```

**Evidence:** `src/handlers/Attack.cpp`

#### ATTACK, PLAYER (11, 8)
**Direction:** Server → Client  
**Purpose:** Broadcast attack animation

**Server Code:**
```cpp
PacketBuilder builder(PACKET_ATTACK, PACKET_PLAYER, 14);
builder.AddShort(attacker_id);
builder.AddChar(direction);
builder.AddShort(target_id);    // 0 if no target
builder.AddThree(damage);
builder.AddThree(target_hp_percent);
builder.AddShort(attacker_tp);  // After attack
builder.AddShort(target_tp);    // After damage
builder.AddChar(0);             // ?
```

**Arena Relevance:** ⭐⭐⭐⭐⭐
- Kill tracking
- Damage calculation
- Combat logs
- Match-ending condition

**Evidence:** `src/map.cpp:1487-1650`

---

### 4. PACKET_CAST (44) - Targeted Spells

#### CAST, REQUEST (44, 1)
**Direction:** Client → Server  
**Purpose:** Cast spell on target

**Client Data:**
```cpp
short spell_id;
short target_id;  // Or 0 for no target
byte timestamp;
```

**Evidence:** `src/handlers/Spell.cpp`

#### CAST, TARGET_OTHER (44, 31)
**Direction:** Server → Client  
**Purpose:** Broadcast spell cast

**Server Code:**
```cpp
PacketBuilder builder(PACKET_CAST, PACKET_TARGET_OTHER, size);
builder.AddShort(caster_id);
builder.AddShort(spell_id);
builder.AddShort(target_id);
builder.AddShort(damage_or_heal);
builder.AddThree(caster_hp_percent);
builder.AddShort(caster_tp);
// ... additional spell effects
```

**Arena Relevance:** ⭐⭐⭐⭐⭐
- Spell-based combat
- Heal tracking
- Buff/debuff management
- Combat logs

---

### 5. PACKET_WARP (19) - Map Transitions

#### WARP, REQUEST (19, 1)
**Direction:** Server → Client  
**Purpose:** Teleport player to map

**Server Code:**
```cpp
PacketBuilder builder(PACKET_WARP, PACKET_REQUEST, size);
builder.AddChar(animation);  // WARP_ANIMATION_*
// Map data follows (EMF)
```

**Arena Relevance:** ⭐⭐⭐⭐⭐
- Arena entry
- Arena exit
- Match teleportation
- Respawn location

**Evidence:** `src/character.cpp:1286-1380`

**Warp Animations:**
```cpp
enum WarpAnimation {
    WARP_ANIMATION_NONE = 0,
    WARP_ANIMATION_SCROLL = 1,    // Default map entry
    WARP_ANIMATION_ADMIN = 2      // Admin warp (bubbles)
};
```

**Evidence:** `src/eodata.hpp:484-489`

---

### 6. PACKET_BOARD (43) - Message Boards

#### BOARD, OPEN (43, 13)
**Direction:** Server → Client  
**Purpose:** Display message board

**Server Code:**
```cpp
PacketBuilder builder(PACKET_BOARD, PACKET_OPEN, estimated_size);
builder.AddChar(board_id + 1);
builder.AddChar(num_posts);

for (each post) {
    builder.AddShort(post_id);
    builder.AddByte(255);
    builder.AddBreakString(author);
    builder.AddBreakString(subject);
}
```

**Arena Relevance:** ⭐⭐⭐⭐⭐
- Leaderboards
- Match history
- Tournament brackets

**Evidence:** `src/character.cpp:1402-1459`, `src/handlers/Board.cpp`

#### BOARD, TAKE (43, 9)
**Direction:** Client → Server  
**Purpose:** Request post content

**Client Data:**
```cpp
short board_id;
short post_id;
```

#### BOARD, PLAYER (43, 8)
**Direction:** Server → Client  
**Purpose:** Send post body

**Server Code:**
```cpp
PacketBuilder reply(PACKET_BOARD, PACKET_PLAYER, 2 + body.length());
reply.AddShort(post_id);
reply.AddString(body);
```

**Evidence:** `src/handlers/Board.cpp:153-169`

#### BOARD, CREATE (43, 6)
**Direction:** Client → Server  
**Purpose:** Create new post

**Client Data:**
```cpp
short board_id;
byte 255;  // separator
break_string subject;
break_string body;
```

**Evidence:** `src/handlers/Board.cpp:55-133`

---

### 7. PACKET_QUEST (50) - Quest/Dialog System

#### QUEST, USE (50, 10)
**Direction:** Client → Server  
**Purpose:** Talk to quest NPC

**Client Data:**
```cpp
short npc_index;
short quest_id;
```

**Evidence:** `src/handlers/Quest.cpp:103-147`

#### QUEST, DIALOG (50, 34)
**Direction:** Server → Client  
**Purpose:** Display quest dialog

**Server Code:**
```cpp
PacketBuilder reply(PACKET_QUEST, PACKET_DIALOG, size);
reply.AddChar(dialog_count);
reply.AddShort(vendor_id);
reply.AddShort(current_quest_id);
reply.AddShort(0);  // session
reply.AddShort(0);  // dialog id
reply.AddByte(255);

// Quest list
for (each quest) {
    reply.AddShort(quest_id);
    reply.AddBreakString(quest_name);
}

// Dialog content (text + links)
dialog->BuildPacket(reply);
```

**Arena Relevance:** ⭐⭐⭐⭐⭐
- Arena menus
- Interactive UI
- Match briefings

**Evidence:** `src/handlers/Quest.cpp:60-90`

#### QUEST, ACCEPT (50, 2)
**Direction:** Client → Server  
**Purpose:** Respond to dialog choice

**Client Data:**
```cpp
short session;
short dialog_id;
short quest_id;
short npc_index;
byte reply_type;  // DIALOG_REPLY_*
byte action;      // Link ID if DIALOG_REPLY_LINK
```

**Evidence:** `src/handlers/Quest.cpp:164-194`

---

### 8. PACKET_MESSAGE (35) - Messages & Dialogs

#### MESSAGE, OPEN (35, 13)
**Direction:** Server → Client  
**Purpose:** Status message popup

**Server Code:**
```cpp
PacketBuilder builder(PACKET_MESSAGE, PACKET_OPEN, message.length());
builder.AddString(message);
```

**Arena Relevance:** ⭐⭐⭐⭐⭐
- Match notifications
- Cooldown warnings
- Error messages

**Evidence:** `src/character.cpp:600-608`

#### MESSAGE, ACCEPT (35, 2)
**Direction:** Server → Client  
**Purpose:** Info box dialog

**Server Code:**
```cpp
PacketBuilder builder(PACKET_MESSAGE, PACKET_ACCEPT, 2 + title.length() + content.length());
builder.AddBreakString(title);
builder.AddString(content);
```

**Arena Relevance:** ⭐⭐⭐⭐⭐
- Match reports
- Statistics display
- Combat logs

**Evidence:** `src/character.cpp:2189-2196`

---

### 9. PACKET_TALK (18) - Chat System

#### TALK, SERVER (18, 23)
**Direction:** Server → Client  
**Purpose:** Server message in chat

**Server Code:**
```cpp
PacketBuilder builder(PACKET_TALK, PACKET_SERVER, message.length());
builder.AddString(message);
```

**Arena Relevance:** ⭐⭐⭐⭐
- Match announcements
- Kill notifications
- Global events

**Evidence:** `src/world.cpp:1248-1258`

#### TALK, MSG (18, 15)
**Direction:** Server → Client  
**Purpose:** Local chat message

**Server Code:**
```cpp
PacketBuilder builder(PACKET_TALK, PACKET_MSG, 3 + name.length() + message.length());
builder.AddBreakString(name);
builder.AddString(message);
```

**Arena Relevance:** ⭐⭐⭐
- Arena-local chat
- Spectator chat

---

### 10. PACKET_SHOP (13) - Shop Interface

#### SHOP, OPEN (13, 13)
**Direction:** Server → Client  
**Purpose:** Display shop window

**Server Code:**
```cpp
PacketBuilder reply(PACKET_SHOP, PACKET_OPEN, size);
reply.AddShort(shop_npc_id);
reply.AddBreakString(shop_name);

// Trade items
for (each item) {
    reply.AddShort(item_id);
    reply.AddThree(buy_price);
    reply.AddThree(sell_price);
    reply.AddChar(max_buy_amount);
}
reply.AddByte(255);

// Craft items
for (each recipe) {
    reply.AddShort(result_item_id);
    for (int i = 0; i < 4; i++) {
        reply.AddShort(ingredient_id);
        reply.AddChar(ingredient_amount);
    }
}
reply.AddByte(255);
```

**Arena Relevance:** ⭐⭐⭐
- Reward redemption
- Trophy vendor
- Arena currency

**Evidence:** `src/handlers/Shop.cpp:173-215`

---

### 11. PACKET_RECOVER (42) - HP/TP Recovery

#### RECOVER, PLAYER (42, 8)
**Direction:** Server → Client  
**Purpose:** Player HP recovery

**Server Code:**
```cpp
PacketBuilder builder(PACKET_RECOVER, PACKET_PLAYER, 7);
builder.AddShort(healer_id);  // Or 0 for natural regen
builder.AddShort(target_id);
builder.AddShort(heal_amount);
builder.AddChar(hp_percent);
```

**Arena Relevance:** ⭐⭐⭐⭐
- Respawn healing
- Potion usage
- Combat tracking

**Evidence:** `src/character.hpp`, `src/map.cpp`

---

### 12. PACKET_REFRESH (25) - Entity Updates

#### REFRESH, REQUEST (25, 1)
**Direction:** Server → Client  
**Purpose:** Full entity state update

**Arena Relevance:** ⭐⭐⭐
- Sync match state
- Reconnection handling
- Spectator join

**Evidence:** `src/handlers/Refresh.cpp`

---

### 13. PACKET_EFFECT (31) - Visual Effects

#### EFFECT, ADMIN (31, 17)
**Direction:** Server → Client  
**Purpose:** Display effect on map

**Server Code:**
```cpp
PacketBuilder builder(PACKET_EFFECT, PACKET_ADMIN, 3);
builder.AddChar(effect_id);
builder.AddShort(target_id);
```

**Arena Relevance:** ⭐⭐⭐
- Victory effects
- Elimination effects
- Power-up effects

**Evidence:** `src/character.cpp:813-830`

---

### 14. PACKET_ITEM (14) - Item Operations

#### ITEM, USE (14, 10)
**Direction:** Client → Server  
**Purpose:** Use item from inventory

**Client Data:**
```cpp
short item_id;
```

**Arena Relevance:** ⭐⭐⭐⭐
- Potion usage
- Equipment changes
- Arena tokens

**Evidence:** `src/handlers/Item.cpp`

---

### 15. PACKET_PARTY (24) - Party System

#### PARTY, REQUEST (24, 1)
**Direction:** Client → Server  
**Purpose:** Party invitation

**Arena Relevance:** ⭐⭐⭐
- Team arena formation
- 2v2, 3v3 modes

**Evidence:** `src/handlers/Party.cpp`

---

### 16. PACKET_EMOTE (9) - Emotes

#### EMOTE, PLAYER (9, 8)
**Direction:** Server → Client  
**Purpose:** Broadcast emote

**Server Code:**
```cpp
PacketBuilder builder(PACKET_EMOTE, PACKET_PLAYER, 4);
builder.AddShort(player_id);
builder.AddChar(emote_id);
```

**Arena Relevance:** ⭐⭐
- Victory celebrations
- Taunts

**Evidence:** `src/character.cpp:799-811`

---

## Packet Flow Examples

### Arena Entry Flow

```
1. Client → Server: QUEST, USE (Arena Master NPC)
2. Server → Client: QUEST, DIALOG (Arena menu)
3. Client → Server: QUEST, ACCEPT (Choose "Enter Queue")
4. Server → Client: MESSAGE, OPEN ("Added to queue...")
5. [Matchmaking finds opponent]
6. Server → Client: MESSAGE, OPEN ("Match found!")
7. Server → Client: WARP, REQUEST (Teleport to arena)
8. Server → Client: REFRESH, REQUEST (Full arena state)
```

### Combat Flow

```
1. Client → Server: ATTACK, USE (Attack direction)
2. Server validates hit
3. Server → All: ATTACK, PLAYER (Broadcast attack + damage)
4. If kill:
   a. Server → All: RECOVER, PLAYER (Respawn victim)
   b. Server → All: WARP, REQUEST (Respawn location)
   c. Server → All: TALK, SERVER ("PlayerA killed PlayerB!")
5. If match end:
   a. Server → All: MESSAGE, ACCEPT (Match report)
   b. Server → All: WARP, REQUEST (Exit arena)
```

### Leaderboard View Flow

```
1. Client → Server: BOARD, OPEN (Board tile interaction)
2. Server → Client: BOARD, OPEN (Ranking list)
3. Client → Server: BOARD, TAKE (Click rank entry)
4. Server → Client: BOARD, PLAYER (Player stats detail)
```

---

## Packet State Requirements

**Evidence:** `src/handlers/handlers.hpp:50-63`

### State Flags

| State | Value | Meaning |
|-------|-------|---------|
| None | 0 | Invalid |
| Uninitialized | 1 | Encryption initialized |
| Menu | 2 | Login screen |
| Character_Menu | 4 | Character select |
| Logging_In | 8 | Entering world |
| Playing | 16 | In-game |
| OutOfBand | 32 | Doesn't queue (flag) |

### Arena Packet States

Most arena packets require `Playing` state:
- PACKET_WALK: Playing
- PACKET_ATTACK: Playing
- PACKET_CAST: Playing
- PACKET_WARP: Any (can warp from menu)
- PACKET_BOARD: Playing
- PACKET_QUEST: Playing

---

## Packet Timing & Rate Limiting

### Attack Delay
**Config:** `SpikeDamageDelay` (milliseconds)  
**Default:** 500ms  
**Purpose:** Prevent attack spam

### Walk Speed
**Config:** `WalkSpeed` (tiles per second)  
**Purpose:** Movement rate limiting

### Spell Cooldown
**Data:** ESF spell cast time  
**Purpose:** Spell spam prevention

**Arena Considerations:**
- Adjust delays for faster-paced combat
- Add arena-specific cooldowns
- Rate limit status queries

---

## Unknown/Undocumented Packets

### PACKET_ARENA (45) - Legacy System

**Status:** ⚠️ LIMITED DOCUMENTATION

The PACKET_ARENA family exists in the protocol but has minimal implementation in current ArenaServ codebase. Legacy arena system used this family.

**Files:** `src/arena.cpp`, `src/arena.hpp`

**Known Issues:**
- Incomplete packet handlers
- Reentrancy issues in state management
- Limited client documentation

**Recommendation:** Use alternative packet families (QUEST, BOARD, MESSAGE) for new arena features rather than extending PACKET_ARENA.

---

## Protocol References

### External Documentation
- **EO Protocol Spec:** `references/eo-protocol-master/` (XML format)
- **Client Reference:** `references/Client/Endless.exe`
- **Alternative Implementation:** `references/EOSource-Revived-main/`

### ArenaServ Source Files
- **Packet Definitions:** `src/fwd/packet.hpp`
- **Packet Handlers:** `src/handlers/*.cpp`
- **Packet Building:** `src/packet.hpp`, `src/packet.cpp`
- **Handler Registration:** `src/handlers/handlers.hpp`, `src/handlers/handlers.cpp`

---

## Summary: Arena Development Strategy

### High-Priority Packets
1. **PACKET_QUEST** - Primary UI system
2. **PACKET_BOARD** - Leaderboards/history
3. **PACKET_MESSAGE** - Notifications/reports
4. **PACKET_WARP** - Arena transitions
5. **PACKET_ATTACK/CAST** - Combat tracking

### Medium-Priority Packets
6. **PACKET_SHOP** - Rewards
7. **PACKET_TALK** - Announcements
8. **PACKET_RECOVER** - Respawn mechanics
9. **PACKET_PARTY** - Team modes
10. **PACKET_REFRESH** - State synchronization

### Low-Priority Packets
11. **PACKET_EFFECT** - Visual polish
12. **PACKET_EMOTE** - Player expression
13. **PACKET_BOOK** - Profile display

### Avoid
- **PACKET_ARENA** - Legacy system, incomplete
- Custom packet families (not supported by client)

---

**Key Insight:** The vanilla EO protocol provides sufficient packet diversity to implement comprehensive arena features without protocol modification. Focus on creative use of existing packets rather than extending the protocol.

