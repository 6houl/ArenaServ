# Window System Capabilities

**Status:** CONFIRMED  
**Last Updated:** 2026-09-22  
**Purpose:** Deep-dive analysis of reusable window systems for arena features

## Overview

The vanilla EO client has several sophisticated window systems that can be repurposed for arena features. This document provides implementation-level details for each system.

---

## 1. Message Board System

**Primary Use:** Town bulletin boards  
**Arena Potential:** ⭐⭐⭐⭐⭐ (Excellent for leaderboards, match history)

### Capabilities

#### Display Format
- **Board ID:** 1-8 (corresponds to Board1-Board8 tile specs)
- **Posts:** List of entries with:
  - Post ID (short, unique per board)
  - Author name (break string)
  - Subject line (break string)
  - Timestamp (optional, formatted as "time ago")
  - Body content (stored separately, retrieved on click)

#### Configuration
```ini
[Board]
MaxUserPosts = 10              # Total posts per user
MaxUserRecentPosts = 3         # Recent posts per user (within RecentPostTime)
RecentPostTime = 86400         # Seconds (24h)
MaxPosts = 50                  # Total posts on board
AdminBoardLimit = 100          # Posts on admin board
BoardMaxSubjectLength = 30     # Subject character limit
BoardMaxPostLength = 2000      # Body character limit
DatePosts = True               # Show timestamps
AdminBoard = 1                 # Admin board ID
```

**Evidence:** `src/character.cpp:1402-1459`, `config/boards.ini`

### Packet Structure

#### Opening Board (PACKET_BOARD, PACKET_OPEN)
```cpp
PacketBuilder builder(PACKET_BOARD, PACKET_OPEN, estimated_size);
builder.AddChar(board_id + 1);                    // 1-8
builder.AddChar(num_posts);                       // Number of posts to display

for (each post) {
    builder.AddShort(post_id);                    // Unique post ID
    builder.AddByte(255);                         // Separator
    builder.AddBreakString(author + author_extra); // Author name
    builder.AddBreakString(subject + subject_extra); // Subject + timestamp
}
```

**Author Extra:**
- If user can post more: `" "` (space) - enables "New Post" button
- Otherwise: `""` (empty) - disables button

**Subject Extra:**
- If `BoardDatePosts` enabled: `" (X days ago)"` format
- Uses `util::timeago(post_time, current_time)`

**Evidence:** `src/character.cpp:1402-1459`

#### Reading Post (PACKET_BOARD, PACKET_PLAYER)
**Client Request:**
```cpp
// Client sends when clicking post
PACKET_BOARD, PACKET_TAKE
short board_id
short post_id
```

**Server Response:**
```cpp
PacketBuilder reply(PACKET_BOARD, PACKET_PLAYER, 2 + body.length());
reply.AddShort(post_id);
reply.AddString(body);  // Full post body content
```

**Evidence:** `src/handlers/Board.cpp:153-169`

#### Creating Post (PACKET_BOARD, PACKET_CREATE)
**Client Request:**
```cpp
PACKET_BOARD, PACKET_CREATE
short board_id
byte 255  // separator
break_string subject
break_string body
```

**Server Response:**
- No explicit reply packet
- Server re-sends board listing (calls `character->ShowBoard()`)

**Validation:**
- Subject capped at `BoardMaxSubjectLength`
- Body capped at `BoardMaxPostLength`
- Checks `MaxUserPosts` limit
- Checks `MaxUserRecentPosts` limit
- Replaces 0xFF bytes with 'y' in subject

**Evidence:** `src/handlers/Board.cpp:55-133`

#### Deleting Post (PACKET_BOARD, PACKET_REMOVE)
**Requirements:**
- Admin level >= `admin_config["boardmod"]`
- OR post author matches character name
- Cannot delete posts from higher-level admins

**Evidence:** `src/handlers/Board.cpp:24-53`

### Arena Applications

#### 1. Global Leaderboard
```cpp
// Example: Top 10 Arena Rankings
Post 1: Author="Rank 1", Subject="PlayerName - 2150 ELO (50-5)"
Post 2: Author="Rank 2", Subject="OtherPlayer - 2080 ELO (45-10)"
...
```

**Body Content (on click):**
```
=== PlayerName's Arena Record ===

Total Matches: 55
Wins: 50
Losses: 5
Win Rate: 90.9%

Current Streak: 12 wins
Best Streak: 15 wins

ELO Rating: 2150
Peak ELO: 2180 (2 days ago)

Recent Matches:
  W vs OtherPlayer (1800) +15 ELO
  W vs Someone (1950) +18 ELO
  L vs TopPlayer (2200) -12 ELO
```

#### 2. Match History Log
```cpp
// Board for player's personal match history
Post 1: Author="Today", Subject="Victory vs PlayerName (1850)"
Post 2: Author="Yesterday", Subject="Defeat vs OtherGuy (1920)"
```

**Body Content:**
```
=== Match #12345 ===
Date: 2026-09-22 14:35

Winner: PlayerName (1850 → 1868)
Loser: OpponentName (1845 → 1827)

Duration: 3:42
Kills: 5 vs 3
Map: Arena_Colosseum

Details:
PlayerName dealt 1250 damage
OpponentName dealt 890 damage
First blood by PlayerName at 0:15
```

#### 3. Tournament Bracket Display
```cpp
// Bracket visualization using ASCII art
Post 1: Author="Finals", Subject="PlayerA vs PlayerB - TBD"
Post 2: Author="Semi-Finals", Subject="4 players remain"
```

**Body Content:**
```
==== TOURNAMENT BRACKET ====

FINALS (Best of 5)
  [PlayerA] ----+
                |---- TBD
  [PlayerB] ----+

SEMI-FINALS (Best of 3)
  PlayerA (W 2-0 vs PlayerC)
  PlayerB (W 2-1 vs PlayerD)

QUARTER-FINALS
  PlayerA (W vs PlayerE)
  PlayerB (W vs PlayerF)
  PlayerC (W vs PlayerG)
  PlayerD (W vs PlayerH)
```

#### 4. Arena News/Updates
```cpp
Post 1: Author="Admin", Subject="New Arena Map Released!"
Post 2: Author="System", Subject="Season 3 Ending Soon"
```

### Limitations
- **Fixed Layout:** Cannot customize window appearance
- **Tile-Based:** Must be triggered by Board tile OR virtual board (needs testing)
- **Subject Length:** 30 chars default (configurable but client UI constrained)
- **No Rich Formatting:** Plain text only
- **Linear List:** No columns, no sorting by client

### Workarounds
- **Virtual Board:** Create board without requiring tile (needs testing)
- **Format Subjects:** Use spacing/symbols for visual structure: `"#1  PlayerName    2150 ELO"`
- **Timestamp Reuse:** Leverage timestamp field for rankings: `"(Rank #1)"` instead of time
- **Body as Data:** Detailed stats in expandable body

---

## 2. Quest Dialog System

**Primary Use:** NPC quest conversations  
**Arena Potential:** ⭐⭐⭐⭐⭐ (Excellent for menus, interactive UI)

### Capabilities

#### Dialog Structure
- **Multi-page dialogs:** Multiple quest dialogs in one interaction
- **NPC Portrait:** Vendor ID determines portrait image
- **Quest List:** Shows active/completed quests
- **Links:** Interactive choices (numbered actions)
- **State Machine:** Quest progression with states

#### Dialog Elements
- **Text Content:** Unlimited length dialog text
- **Links:** Numbered choices (action IDs)
- **Quest Name:** Identifying quest in list
- **Multiple Vendors:** Same quest, different NPCs, different dialogs

### Packet Structure

#### Opening Quest Dialog (PACKET_QUEST, PACKET_DIALOG)
```cpp
PacketBuilder reply(PACKET_QUEST, PACKET_DIALOG, estimated_size);
reply.AddChar(dialog_count);              // Number of available quests
reply.AddShort(vendor_id);                // NPC ID for portrait
reply.AddShort(current_quest_id);         // Quest being displayed
reply.AddShort(0);                        // Session ID (unused)
reply.AddShort(0);                        // Dialog ID (unused)
reply.AddByte(255);                       // Separator

// Quest list
for (each available_quest) {
    reply.AddShort(quest_id);
    reply.AddBreakString(quest_name);
}

// Current dialog content
dialog->BuildPacket(reply);  // Adds text and links
```

**Evidence:** `src/handlers/Quest.cpp:60-90`

#### Dialog Content Building
From `Dialog` class (src/dialog.hpp, src/eoplus.cpp):
```cpp
// Dialog text pages
builder.AddShort(num_pages);
for (each page) {
    builder.AddBreakString(page_text);
}

// Dialog links (choices)
builder.AddShort(num_links);
for (each link) {
    builder.AddShort(link_action_id);
    builder.AddBreakString(link_text);
}
```

**Link Actions:**
- Numeric IDs (1-255)
- Server handles via `quest->DialogInput(action)`
- Can trigger state changes, warps, item rewards, etc.

**Evidence:** `src/handlers/Quest.cpp:164-194`

#### Quest List View (PACKET_QUEST, PACKET_LIST)
```cpp
PacketBuilder reply(PACKET_QUEST, PACKET_LIST, estimated_size);
reply.AddChar(page);  // QUEST_PAGE_PROGRESS (0) or QUEST_PAGE_HISTORY (1)
reply.AddShort(quest_count);

for (each quest) {
    // Format depends on page
    // Progress page: quest name + current objective
    // History page: quest name + completion date
}
```

**Evidence:** `src/handlers/Quest.cpp:256-300`

### Arena Applications

#### 1. Arena Master Menu
```cpp
// Quest Dialog as Main Menu
vendor_id = 100 (Arena Master NPC)
quest_id = 9001 (Arena System)

Dialog Text:
"Welcome to the Arena! What would you like to do?"

Links (Choices):
  1 → "Enter Matchmaking Queue"
  2 → "View My Statistics"
  3 → "View Leaderboards"
  4 → "Arena Shop (Rewards)"
  5 → "Rules & Information"
  6 → "Exit"
```

**Implementation:**
```cpp
// In arena quest file (data/quests/arena.eqf)
state Begin
    dialog {
        text "Welcome to the Arena! What would you like to do?"
        link 1 "Enter Matchmaking Queue"
        link 2 "View My Statistics"
        link 3 "View Leaderboards"
        link 4 "Arena Shop"
        link 5 "Rules"
        link 6 "Exit"
    }
    
    // Handle choices
    rule LinkAction 1 {
        // Add to queue logic
        SetState("InQueue")
    }
    
    rule LinkAction 2 {
        // Show stats dialog
        SetState("ShowStats")
    }
    // ... etc
end_state
```

#### 2. Statistics Display
```cpp
// After choosing "View Stats"
Dialog Text:
"=== Your Arena Statistics ===\n
Current Rank: #42\n
ELO Rating: 1850\n\n
Total Matches: 120\n
Wins: 75 (62.5%)\n
Losses: 45 (37.5%)\n\n
Current Streak: 3 wins\n
Best Streak: 12 wins\n\n
Total Kills: 450\n
K/D Ratio: 1.8\n\n
Highest ELO: 1920\n
Average Match Time: 4:35"

Links:
  1 → "View Match History"
  2 → "Back to Menu"
```

#### 3. Match Briefing
```cpp
// Pre-match information
Dialog Text:
"Match Found!\n\n
Opponent: PlayerName\n
Opponent ELO: 1800\n
Opponent Record: 45-30\n\n
Map: Arena_Colosseum\n
Mode: 1v1 Deathmatch\n\n
Victory: +18 ELO\n
Defeat: -15 ELO\n\n
Prepare yourself!"

Links:
  1 → "I'm Ready!" (Enter arena)
  2 → "Forfeit" (Leave queue)
```

#### 4. Interactive Tutorials
```cpp
// Multi-page tutorial
Page 1:
"Arena Basics - Movement\n
Use arrow keys to move...\n
(Tutorial content)"

Page 2:
"Arena Basics - Combat\n
Press SPACEBAR to attack...\n
(Tutorial content)"

Links:
  1 → "Next Page"
  2 → "Previous Page"
  3 → "Skip Tutorial"
```

#### 5. Achievement Browser
```cpp
// Using quest list for achievements
Quest List (QUEST_PAGE_PROGRESS):
  Quest 8001: "First Victory (In Progress)"
  Quest 8002: "10 Wins (5/10)"
  Quest 8003: "50 Kills (50/50 - Claim Reward!)"

// Click to expand
Dialog Text:
"=== Achievement: 50 Kills ===\n\n
Congratulations! You've eliminated\n
50 opponents in the arena.\n\n
Reward: 500 Arena Points\n
Reward: Title 'Slayer'\n\n
Click below to claim your reward!"

Links:
  1 → "Claim Reward"
  2 → "Back"
```

### Limitations
- **NPC Required:** Must interact with ENF NPC (vendor_id)
- **Portrait Limited:** Portrait determined by vendor_id (fixed sprites)
- **Text Only:** No graphics, icons, or colors
- **Link Limit:** Maximum ~255 links (practical limit lower)
- **No Input Fields:** Cannot enter custom text (only link choices)

### Workarounds
- **Generic Arena NPC:** Create one "Arena Master" NPC type, spawn everywhere
- **Text Formatting:** Use newlines, spacing, and ASCII symbols for structure
- **State Machine:** Complex menus via quest states (sub-menus)
- **Quest IDs as Categories:** Different quest IDs for different menu categories

---

## 3. Info Box System

**Primary Use:** Admin commands, multi-line messages  
**Arena Potential:** ⭐⭐⭐⭐ (Great for detailed stats, logs)

### Capabilities

#### Display Format
- **Title Bar:** String title (modal dialog header)
- **Content Area:** Scrollable text body
- **Automatic Word Wrap:** 197 pixel width
- **Unlimited Length:** Content can be very long
- **Modal Dialog:** Blocks other actions until closed

#### Word Wrap Algorithm
```cpp
constexpr size_t INFOBOX_WIDTH = 197;  // pixels
constexpr size_t SPACE_WIDTH = 3;       // pixels

// Intelligent word wrapping
// - Preserves leading whitespace
// - Wraps on word boundaries
// - Adds padding spaces to fill lines
// - Handles empty lines
```

**Evidence:** `src/character.cpp:2068-2189`

### Packet Structure

#### Simple Version
```cpp
PacketBuilder builder(PACKET_MESSAGE, PACKET_ACCEPT, 2 + title.length() + content.length());
builder.AddBreakString(title);
builder.AddString(content);  // Manual formatting with spacing
character->Send(builder);
```

#### Advanced Version (Automatic Formatting)
```cpp
// Character::ShowInfoBox(title, vector<string> lines)
// Handles word wrap automatically
// Adds padding spaces to fill 197px width
// Preserves line breaks between vector entries

std::vector<std::string> lines = {
    "Line 1 content here",
    "Line 2 with automatic wrapping if it's too long",
    "",  // Empty line
    "Line 4"
};

character->ShowInfoBox("Title", lines);
```

**Evidence:** `src/character.cpp:2067-2189`

### Arena Applications

#### 1. Detailed Match Report
```cpp
character->ShowInfoBox("Match #12345 Results", {
    "=== VICTORY! ===",
    "",
    "Winner: " + winner_name + " (" + winner_elo_before + " → " + winner_elo_after + ")",
    "Loser: " + loser_name + " (" + loser_elo_before + " → " + loser_elo_after + ")",
    "",
    "Duration: " + duration,
    "Total Kills: " + kills,
    "",
    "=== Statistics ===",
    winner_name + ":",
    "  Damage Dealt: " + winner_damage,
    "  Damage Taken: " + winner_damage_taken,
    "  Healing: " + winner_healing,
    "  Accuracy: " + winner_accuracy + "%",
    "",
    loser_name + ":",
    "  Damage Dealt: " + loser_damage,
    "  Damage Taken: " + loser_damage_taken,
    "  Healing: " + loser_healing,
    "  Accuracy: " + loser_accuracy + "%",
    "",
    "=== Rewards ===",
    "ELO: +" + elo_gain,
    "Arena Points: +" + points_gain,
    "Experience: +" + exp_gain,
});
```

#### 2. Combat Log Viewer
```cpp
character->ShowInfoBox("Match Combat Log", {
    "=== Combat Log ===",
    "",
    "0:05 - PlayerA attacked for 45 damage",
    "0:08 - PlayerB used Health Potion (+100 HP)",
    "0:12 - PlayerA cast Fire Spell (85 damage)",
    "0:15 - PlayerB attacked for 52 damage",
    "0:20 - PlayerA attacked for 48 damage",
    "0:25 - PlayerB cast Ice Spell (MISS)",
    "0:30 - PlayerA attacked for 50 damage",
    "0:33 - PlayerB eliminated",
    "",
    "Total Duration: 0:33",
    "Winner: PlayerA",
});
```

#### 3. Rules & Help System
```cpp
character->ShowInfoBox("Arena Rules", {
    "=== ARENA RULES ===",
    "",
    "1. MATCHMAKING",
    "   - Players matched by ELO rating",
    "   - ±200 ELO range",
    "   - Wait time: up to 5 minutes",
    "",
    "2. COMBAT",
    "   - First to 5 kills wins",
    "   - Respawn time: 5 seconds",
    "   - Match time limit: 10 minutes",
    "",
    "3. EQUIPMENT",
    "   - All gear allowed",
    "   - Potions: max 10 per match",
    "   - Stat foods: prohibited",
    "",
    "4. ELO SYSTEM",
    "   - Starting ELO: 1200",
    "   - Win: +10 to +30 ELO (opponent-dependent)",
    "   - Loss: -10 to -30 ELO",
    "   - Draw: ±5 ELO",
    "",
    "5. REWARDS",
    "   - Win: 50-150 Arena Points",
    "   - Loss: 10 Arena Points (participation)",
    "   - Daily Bonus: First 3 wins +50%",
    "",
    "For assistance, contact an Arena Master.",
});
```

#### 4. Player Profile Summary
```cpp
character->ShowInfoBox(target_name + "'s Arena Profile", {
    "=== " + target_name + " ===",
    "",
    "Rank: #" + rank,
    "ELO: " + elo + " (" + tier + ")",
    "Title: " + arena_title,
    "",
    "=== Statistics ===",
    "Total Matches: " + total_matches,
    "Win Rate: " + win_rate + "% (" + wins + "W - " + losses + "L)",
    "Current Streak: " + streak_type + " " + streak_count,
    "Best Streak: " + best_streak + " wins",
    "",
    "=== Performance ===",
    "Average K/D: " + kd_ratio,
    "Total Kills: " + total_kills,
    "Average Match Time: " + avg_match_time,
    "Fastest Victory: " + fastest_win,
    "",
    "=== Achievements ===",
    achievement_list,  // "- First Blood Master\n- Untouchable (20 wins)"
    "",
    "=== Recent Matches ===",
    recent_match_1,  // "W vs PlayerX (1820) +15 ELO"
    recent_match_2,
    recent_match_3,
});
```

### Limitations
- **Modal Dialog:** Blocks all actions until closed
- **No Interactivity:** Cannot have buttons/links (read-only)
- **Text Only:** No colors, icons, or graphics
- **Fixed Width:** 197 pixels (~35-40 characters depending on font)
- **Manual Close:** User must click close button

### Workarounds
- **ASCII Art:** Use spacing for tables/structure
- **Line Breaks:** Strategic spacing for readability
- **Symbols:** Unicode/ASCII symbols for bullets, separators
- **Pagination:** Multiple info boxes for very long content (chain with quest dialogs)

---

## 4. Shop System

**Primary Use:** NPC vendors  
**Arena Potential:** ⭐⭐⭐ (Good for rewards, creative stat display)

### Capabilities

#### Shop Structure
- **Shop Name:** String identifier
- **Trade Items:** Buy/sell items with prices
- **Craft Items:** Crafting recipes (ingredients → result)
- **Tabs:** Buy, Sell, Craft (client-controlled)
- **Currency Display:** Shows player's "gold" (can repurpose)

#### Item Requirements
- **Must exist in EIF:** All items shown must be in EIF file
- **Icons:** Item graphics from EIF
- **Names:** Item names from EIF
- **Descriptions:** Item descriptions from EIF

### Packet Structure

#### Opening Shop (PACKET_SHOP, PACKET_OPEN)
```cpp
PacketBuilder reply(PACKET_SHOP, PACKET_OPEN, estimated_size);
reply.AddShort(shop_npc_id);
reply.AddBreakString(shop_name);

// Trade items (buy/sell)
UTIL_FOREACH(trade_items, item) {
    reply.AddShort(item_id);       // From EIF
    reply.AddThree(buy_price);     // Cost to buy (gold or "arena points")
    reply.AddThree(sell_price);    // Value when selling
    reply.AddChar(max_buy_amount); // Max per transaction
}
reply.AddByte(255);  // End trade items

// Craft items
UTIL_FOREACH(craft_items, item) {
    reply.AddShort(result_item_id);
    
    // Up to 4 ingredients
    for (int i = 0; i < 4; i++) {
        if (i < item.ingredients.size()) {
            reply.AddShort(ingredient_item_id);
            reply.AddChar(ingredient_amount);
        } else {
            reply.AddShort(0);
            reply.AddChar(0);
        }
    }
}
reply.AddByte(255);  // End craft items

character->Send(reply);
```

**Evidence:** `src/handlers/Shop.cpp:173-215`

#### Shop Data File (data/shops.ini)
```ini
[Shop Name]
Trade = Item1,Item2,Item3
Craft = Recipe1,Recipe2
```

**Evidence:** `data/shops.ini`

### Arena Applications

#### 1. Arena Reward Vendor
```cpp
// Shop: "Arena Rewards"
// Currency: Arena Points (displayed as "gold")

Trade Items:
  - Trophy_Bronze (ID 5001): 100 points
  - Trophy_Silver (ID 5002): 500 points
  - Trophy_Gold (ID 5003): 1000 points
  - Arena_Title_Scroll (ID 5004): 2000 points
  - Cosmetic_Arena_Armor (ID 5005): 5000 points
  - Rare_Mount_Token (ID 5006): 10000 points
```

**Implementation:**
1. Create trophy items in EIF
2. Define shop in `data/shops.ini`
3. Create Arena Vendor NPC (ENF)
4. Custom handler to use "arena points" instead of gold

#### 2. Ranking Visualization (Creative Use)
```cpp
// Shop: "Arena Rankings" (view-only, don't buy)
// Use item names/prices to show rankings

// Create dummy items in EIF:
// - Rank1_Trophy (name: "Rank #1")
// - Rank2_Trophy (name: "Rank #2")
// etc.

Trade Items:
  - Rank1_Trophy: 2150 ELO (buy price = ELO rating)
  - Rank2_Trophy: 2080 ELO
  - Rank3_Trophy: 2010 ELO
  // ... up to top 20

// Item descriptions (in EIF) contain player names
```

**Limitations:** Requires many dummy items, hacky

#### 3. Stat "Crafting" Display (Creative Use)
```cpp
// Shop: "Arena Progression"
// Craft recipes = tier requirements

// Example: "Craft" Bronze → Silver tier
Craft Item: Silver_Arena_Tier
Ingredients:
  - 50x Arena_Win_Token (50 wins)
  - 100x Kill_Counter (100 kills)
  - 1x Bronze_Tier_Badge (current tier)
  - 1200x ELO_Crystal (1200+ ELO)
```

**Limitations:** Very creative/hacky use, confusing UI

### Limitations
- **Requires EIF Items:** Cannot show items not in EIF
- **Fixed Currency:** Shows gold icon (not "Arena Points" text)
- **No Custom UI:** Fixed shop window layout
- **Tab Confusion:** Client controls tabs, might confuse "creative" uses

### Workarounds
- **Item Naming:** Use EIF item names cleverly: "Arena Token (500 points)"
- **Descriptions:** Detailed descriptions in EIF
- **NPCs:** Multiple shop NPCs for categorization
- **Sell-Only Shops:** Disable buying for view-only displays

---

## 5. Book / Character Info

**Primary Use:** Player profiles  
**Arena Potential:** ⭐⭐ (Limited but possible)

### Capabilities

#### Display Fields
- **Name:** Character name
- **Home:** Home town string
- **Partner:** Partner/spouse string
- **Title:** Title string
- **Guild:** Guild name
- **Guild Rank:** Guild rank string
- **Class:** Class icon (from ECF)
- **Gender:** Gender icon
- **Admin:** Admin icon (optional)

### Packet Structure

```cpp
PacketBuilder builder(PACKET_BOOK, PACKET_REPLY, estimated_size);
builder.AddBreakString(name);
builder.AddBreakString(home);
builder.AddBreakString(partner);
builder.AddBreakString(title);
builder.AddBreakString(guild_name);
builder.AddBreakString(guild_rank);
builder.AddChar(class_id);
builder.AddChar(gender);
builder.AddChar(admin_icon);
```

**Evidence:** `src/handlers/Book.cpp:36-105`

### Arena Applications

#### Arena Profile Overlay
```cpp
// Repurpose fields for arena stats
Name: "PlayerName"
Home: "Arena Rank: #42"
Partner: "Current Streak: 12 wins"
Title: "Arena Champion"
Guild: "ELO: 1850"
Guild Rank: "Win Rate: 65%"
```

### Limitations
- **Fixed Fields:** Cannot add new fields
- **String Only:** Limited to text replacements
- **Icons Fixed:** Cannot change class/gender icons
- **Short Length:** Field length limits

---

## System Comparison Table

| System | Interactivity | Length | Scrollable | Best Use |
|--------|--------------|--------|-----------|----------|
| Message Board | ⭐⭐ (posts) | Medium (2000 chars) | ✅ | Leaderboards, logs |
| Quest Dialog | ⭐⭐⭐ (links) | Unlimited | ✅ | Menus, navigation |
| Info Box | ❌ (read-only) | Unlimited | ✅ | Stats, reports |
| Shop | ⭐⭐ (buy/sell) | N/A | ✅ | Rewards, exchanges |
| Book | ❌ (read-only) | Short (~100 chars/field) | ❌ | Quick stats |

---

## Recommended Arena UI Architecture

### Main Entry Point
**Quest Dialog** - Arena Master NPC
- Root menu with links to all features
- State machine navigation

### Feature-Specific Windows

#### Statistics & Reports
**Info Box** - Detailed, read-only stats
- Match reports
- Personal statistics
- Combat logs

#### Rankings & History
**Message Board** - List-based displays
- Global leaderboards
- Match history
- Tournament brackets

#### Navigation
**Quest Dialog** - Interactive menus
- Sub-menus
- Confirmations
- Tutorials

#### Rewards
**Shop** - Item-based exchanges
- Spend arena points
- Purchase rewards
- Trophy collection

---

**Key Finding:** By combining these window systems, nearly all planned arena features can be implemented server-side without client modification.
