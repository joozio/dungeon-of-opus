# Dungeon of Opus — Gameplay Reference

> A complete roguelike built entirely by Claude Opus 4.6 in a single React component.
> **Play it live:** [wiz.jock.pl/experiments/dungeon-of-opus](https://wiz.jock.pl/experiments/dungeon-of-opus)

---

## Overview

Dungeon of Opus is a turn-based roguelike dungeon crawler. You descend through 5 procedurally generated floors, fighting enemies, collecting items, and growing stronger. Reach the bottom floor and survive an encounter with **Opus, the Elder Dragon** to win.

**Core traits:**
- **Permadeath** — when you die, you start over. No saves, no second chances.
- **Turn-based** — the world only moves when you do.
- **Procedural** — every floor is generated fresh each run. No two dungeons are the same.
- **Fog of war** — you only see tiles within line-of-sight (Bresenham's algorithm, radius 7).

**Victory condition:** Reach floor 5, kill Opus the Elder Dragon, and descend the final stairs.

---

## Keyboard Controls

| Key | Action |
|-----|--------|
| `W` / `Arrow Up` | Move north |
| `S` / `Arrow Down` | Move south |
| `A` / `Arrow Left` | Move west |
| `D` / `Arrow Right` | Move east |
| `.` or `5` | Wait one turn (enemies still move) |
| `I` | Open/close inventory |
| `?` | Toggle help screen |
| `Enter` / `Space` | Confirm on title / game over screens |

**On mobile:** A D-pad appears below the dungeon grid with directional buttons, a WAIT button, and an INV button.

**Combat:** Walk into an enemy to attack it. No separate attack command needed.

---

## The Map

The dungeon is rendered as a 40x22 ASCII viewport onto a 48x28 tile map. The camera centers on the player.

### Tile Types

| Symbol | Color | Tile | Notes |
|--------|-------|------|-------|
| `#` | Gray | Wall | Impassable |
| `·` | Dark gray | Floor | Standard movement |
| `+` | Amber | Door | Walkable; opens (converts to floor) on contact |
| `>` | Cyan | Stairs | Descend to next floor |
| `^` | Orange | Trap | Hidden until visible; triggers on step, then disappears |
| `~` | Blue | Water | Passable; prints a slow message |
| `~` | Red | Lava | Deals 5–10 damage per step |
| `$` | Yellow | Gold | Collected automatically on step |

Traps are invisible until you enter line-of-sight. Lava appears from floor 4 onward. Water appears from floor 2 onward.

### Dungeon Generation

Each floor is built by:
1. Placing 6–16 rooms (6 + floor×2 target) of random size (4–9 wide, 3–7 tall) with 2-tile padding between them.
2. Connecting rooms sequentially with L-shaped corridors (randomly horizontal-first or vertical-first).
3. Adding extra random corridors (~1 per 3 rooms) for connectivity.
4. Placing doors (30% chance) at corridor-room boundaries.
5. Scattering traps (2–7 count, scales with floor), water/lava patches, gold, and items.
6. Placing stairs in the center of the last room.

The player always starts in the center of the first room. Stairs always appear in the last room.

---

## The Player

**Starting stats:**
- HP: 30 / 30
- Attack: 3
- Defense: 1
- Inventory: Health Potion + Bread

**Starting inventory** always contains one Health Potion and one Bread for the first floor.

### Stats

| Stat | Effect |
|------|--------|
| HP | Current / max health. Reach 0 and it's game over. |
| Attack | Base damage. Added to weapon bonus before mitigation. |
| Defense | Reduces incoming damage. Added to armor bonus. |
| XP | Earned by killing enemies. Accumulates toward next level. |
| Gold | Collected from floors. Contributes to score. |
| Keys | Used to open locked doors (if any). |
| Food Timer | Hunger meter. Starts at 100, depletes 1 per turn. |

### Leveling Up

XP thresholds: `0, 10, 25, 50, 100, 180, 300, 500, 800, 1200` (max level 20).

On level up:
- Max HP +5
- HP restored by 10 (capped at new max)
- Attack +1
- Defense +1

### Hunger

The food timer depletes 1 per turn. At 0 you start taking 1 HP damage per turn. Status labels:

| Food Timer | Status | Color |
|------------|--------|-------|
| > 60 | Satiated | Green |
| 21–60 | Hungry | Yellow |
| 1–20 | Starving | Red |
| ≤ 0 | Famished | Red |

Eat food items (Bread, Feast) from inventory to restore the timer.

---

## Combat Mechanics

Combat is strictly turn-based. Every time the player acts, all enemies also take a turn.

### Player Attacks an Enemy

**Formula:** `damage = max(1, (player.attack + weapon.value) - floor(enemy.defense / 2))`

- **Critical hit:** 10% chance — damage is doubled.
- XP is awarded equal to the enemy's XP value on kill.
- Score increases by `xp * 10` per kill.

### Enemy Attacks the Player

**Formula:** `damage = max(1, enemy.attack - floor((player.defense + armor.value) / 2))`

- Minimum 1 damage always lands.
- Vampires additionally drain life equal to half the damage dealt (the drain is noted in the log but does not heal the vampire in the current implementation — it only adds flavor text).

---

## Enemy Types

All enemies have their base stats scaled by floor: `stat = floor(base * (1 + (floor - 1) * 0.15))`. A floor 3 enemy has ~1.3x the stats of its floor 1 counterpart.

### Behavior Modes

| Behavior | Description |
|----------|-------------|
| `wander` | Moves randomly (40% chance each turn). Attacks if adjacent. |
| `chase` | Moves toward last known player position when alerted. Loses interest if it reaches the spot without seeing the player. |
| `ranged` | Fires energy bolts at range ≤4 and backs away. Falls back to melee if cornered (distance 1). |
| `ambush` | Stationary until player is within distance 2. Then chases. |
| `guard` | Only activates if player is within distance 4 or has been alerted. |

Enemies become **alerted** when they have line-of-sight to the player within distance 7 (the same as the player's sight range). Once alerted, chasers pursue the last known player position.

---

### Enemy Reference

| # | Name | Icon | Color | HP | Attack | Defense | XP | Behavior | Floors |
|---|------|------|-------|----|--------|---------|-----|----------|--------|
| 1 | Slime | `s` | Green | 8 | 2 | 0 | 3 | wander | 1–2 |
| 2 | Skeleton | `S` | Gray | 15 | 5 | 2 | 8 | chase | 1–3 |
| 3 | Dark Mage | `M` | Purple | 12 | 8 | 1 | 12 | ranged | 2–5 |
| 4 | Mimic | `$` | Yellow | 20 | 7 | 3 | 15 | ambush | 1–5 |
| 5 | Vampire | `V` | Red | 25 | 6 | 3 | 18 | chase | 3–5 |
| 6 | Stone Golem | `G` | Stone | 40 | 9 | 8 | 25 | guard | 3–5 |
| 7 | Elder Dragon | `D` | Bright Red | 80 | 15 | 10 | 100 | chase | 5 |
| — | **Opus, the Elder Dragon** | `D` | Bright Red | **120** | **18** | **12** | **200** | chase | **5 (boss)** |

---

### Enemy Detail: Slime

- **Appearance:** `s` in green.
- **Behavior:** Wanders aimlessly. Attacks if the player steps adjacent.
- **Difficulty:** Trivial. Lowest stats in the game. Essentially free XP.
- **Special:** None. Pure early-game fodder.
- **Floors:** 1–2. Disappears from pools by floor 3.

---

### Enemy Detail: Skeleton

- **Appearance:** `S` in gray.
- **Behavior:** Chaser. Becomes alerted on sight, then pursues the player's last known position. Loses interest if it reaches the spot and can no longer see the player.
- **Difficulty:** Easy. Moderate attack and some defense, but manageable early.
- **Special:** None. The game's baseline chasing enemy.
- **Floors:** 1–3. Dominates floor 1–2 encounters.

---

### Enemy Detail: Dark Mage

- **Appearance:** `M` in purple.
- **Behavior:** Ranged. When within distance 4 with line-of-sight, fires an energy bolt (deals `attack - floor(defense/2)` damage). Simultaneously tries to move away from the player. If forced into melee range (distance 1), attacks normally.
- **Difficulty:** Medium-hard. High attack relative to low HP. Dangerous if multiple appear.
- **Special:** The only enemy with a dedicated ranged attack. It actively kites — it will shoot and retreat simultaneously.
- **Floors:** 2–5.

---

### Enemy Detail: Mimic

- **Appearance:** `$` in yellow (looks identical to gold on the floor). Reveals itself as `$` in yellow once alerted — same icon, slightly different context in the log.
- **Behavior:** Ambush. Stationary, disguised as gold. Does not move or react until the player is within distance 2. Then chases. Also becomes alerted if the player enters distance 2 while it has line-of-sight.
- **Difficulty:** Medium. Nasty surprise on crowded floors. Comparable stats to Vampire.
- **Special:** Disguise — you cannot distinguish it from a gold pile until you're already close.
- **Floors:** All floors (1–5). One of the most consistent threats throughout.

---

### Enemy Detail: Vampire

- **Appearance:** `V` in red.
- **Behavior:** Chaser. Identical pursuit logic to Skeleton but with higher stats.
- **Difficulty:** Hard. Strong HP and attack. Appears from floor 3 onward.
- **Special:** Life drain — every successful attack is logged as "Drains life!" The drain is cosmetic in the message log.
- **Floors:** 3–5. Dominant enemy on floor 4.

---

### Enemy Detail: Stone Golem

- **Appearance:** `G` in stone/muted gray.
- **Behavior:** Guard. Remains stationary unless the player comes within distance 4 or it has been alerted by another means. Once activated, moves toward the player.
- **Difficulty:** Very hard. 40 base HP and 8 base defense make it the tankiest non-boss enemy. The defense mitigation means most early-game attacks deal minimum 1 damage.
- **Special:** Territory guard. Can effectively block key areas of a floor, especially since it appears in the final rooms of later dungeons.
- **Floors:** 3–5. Dominant on floor 4.

---

### Enemy Detail: Elder Dragon (regular)

- **Appearance:** `D` in bright red.
- **Behavior:** Chaser. Same pursuit AI as Skeleton but dramatically stronger.
- **Difficulty:** Extreme. 80 HP, 15 attack, 10 defense. A run-ending encounter if fought unprepared.
- **Special:** Can appear as a regular floor enemy on floor 5 (weight 15). Multiple may spawn.
- **Floors:** Floor 5 only.

---

### Boss: Opus, the Elder Dragon

The guaranteed boss of floor 5. Placed in the center of the final room.

| Stat | Value |
|------|-------|
| HP | 120 |
| Attack | 18 |
| Defense | 12 |
| XP | 200 |
| Behavior | chase |

Defeating Opus unlocks the victory screen. He uses the same chase AI as regular dragons — no phase transitions or special attacks beyond his raw stat advantage. The challenge is surviving long enough to whittle him down while managing hunger, floor hazards, and the other floor 5 enemies.

---

## Floor Progression

| Floor | Rooms | Enemy Count | Enemy Pool | Traps | Hazards | Item Pool |
|-------|-------|-------------|------------|-------|---------|-----------|
| 1 | 8 | 6–12 | Slime 50%, Skeleton 30%, Mimic 20% | 2–4 | None | Basic potions, rusty weapons, bread |
| 2 | 10 | 8–15 | Slime 20%, Skeleton 35%, Mage 25%, Mimic 20% | 2–5 | Water patches | Steel blades, chain mail |
| 3 | 12 | 10–18 | Skeleton 20%, Mage 25%, Vampire 25%, Mimic 15%, Golem 15% | 2–6 | Water patches | Flame swords, plate armor, elixirs |
| 4 | 14 | 12–21 | Mage 20%, Vampire 30%, Golem 30%, Mimic 20% | 2–7 | Lava patches | Rare/legendary starts appearing |
| 5 | 16 | 14–24 + boss | Vampire 25%, Golem 25%, Dragon 15%, Mage 20%, Mimic 15% | 2–8 | Lava patches | Elixirs, legendary items, Opus Blade |

Enemy count formula: `rng(4 + floor*2, 6 + floor*3)`. Gold per pile scales as `rng(1, 5 + floor*3)`. No stairs are placed on floor 5 — you win by killing the boss (the game transitions to Victory when you step on the last staircase tile, which is only reachable after completing floor 5 by having been placed there on the final floor's map).

---

## Items

Items appear on floors as colored ASCII characters. Rarity determines both drop weight and display color.

| Color | Rarity |
|-------|--------|
| White | Common |
| Green | Uncommon |
| Blue | Rare |
| Amber/Gold | Legendary |

### Potions (`!`)

| Name | Rarity | Heal Amount |
|------|--------|-------------|
| Health Potion | Common | 15 HP |
| Greater Potion | Uncommon | 35 HP |
| Elixir of Life | Rare | 60 HP |

Potions restore HP capped at max HP. Cannot overheal.

### Weapons (`)`)

| Name | Rarity | Attack Bonus | First Appears |
|------|--------|-------------|---------------|
| Rusty Sword | Common | +2 | Floor 1 |
| Steel Blade | Uncommon | +4 | Floor 1–2 |
| Flame Sword | Rare | +7 | Floor 2–3 |
| Blade of Opus | Legendary | +12 | Floor 4–5 |

Equipping a weapon automatically unequips the previous one (returned to inventory). Only one weapon at a time. Uses `(` icon, displayed as `)` in the source.

### Armor (`[`)

| Name | Rarity | Defense Bonus | First Appears |
|------|--------|--------------|---------------|
| Leather Armor | Common | +2 | Floor 1 |
| Chain Mail | Uncommon | +4 | Floor 2 |
| Plate Armor | Rare | +7 | Floor 3–4 |
| Mythril Armor | Legendary | +11 | Floor 4–5 |

Same swap logic as weapons. One armor slot.

### Scrolls (`?`)

| Name | Rarity | Effect |
|------|--------|--------|
| Scroll of Fire | Uncommon | Deals 20 damage to all enemies within Manhattan distance 3. XP awarded for kills. |
| Scroll of Teleport | Uncommon | Teleports player to a random position within a random room. |
| Scroll of Reveal | Rare | Marks every tile on the current floor as explored (full map visible in fog). |

### Food (`%`)

| Name | Rarity | Food Timer Restored | Bonus Heal |
|------|--------|--------------------|-----------:|
| Bread | Common | +60 (capped at 200) | +5 HP |
| Feast | Uncommon | +200 (capped at 200) | +5 HP |

Food also heals 5 HP on use (capped at max HP).

### Keys (`k`)

Iron Key — Common. Adds 1 key to your count. Keys display on the equipment panel. (Locked doors are present in the codebase as a mechanic; standard doors are passable by walking.)

---

## Scoring

Score accumulates during a run:

| Source | Points |
|--------|--------|
| Enemy kill | enemy XP × 10 |
| Gold collected | gold amount |
| Victory bonus | gold × 2 + kills × 10 + level × 50 |
| Death score | gold + kills × 5 |

High score is persisted in `localStorage` across sessions.

---

## The HUD

```
Floor X/5   Lv X   HP XX/XX   ATK X+X   DEF X+X   Gold XX   [Hunger Status]
[HP bar ████████░░░░░░░░░░]   [XP bar ██████░░░░░░░░░░░░░░]

[ dungeon viewport 40x22 ]      MAP   (minimap)
                                EQUIPMENT
                                  Weapon: ...
                                  Armor:  ...
                                  Keys:   X
                                NEARBY
                                  (visible enemies + HP)
                                WASD/Arrows | I: Inv | .: Wait
LOG
> last 5 messages
```

The minimap shows explored tiles in dark gray, walls in medium gray, stairs in cyan, water in blue, lava in red-orange, the player as yellow, and visible enemies as red.

---

## Tips

- **Mimics are the most dangerous early threat.** They look exactly like gold. Approach gold carefully on floors 1–2.
- **Dark Mages kite.** They fire and retreat. Chase them into corners or use Scroll of Fire.
- **Stone Golems have extreme defense.** Without a Flame Sword or better, you deal 1 damage per hit. Avoid or use scrolls.
- **Hunger is a timer, not a race.** Food drops are reliable enough. Prioritize it when below 30.
- **The Scroll of Reveal is run-defining.** Finding stairs early saves enormous HP.
- **Lava on floors 4–5 is a genuine hazard.** 5–10 damage per tile walked on. Don't cut corners.
- **Wait (`.`)** when injured and enemies are out of sight — lets you position without advancing hunger faster than necessary.
- **Equip before descending.** Check inventory after every floor clear to swap in better gear.

---

*Built by Claude Opus 4.6 in a single session. 1,668 lines of TypeScript. No game engine. No sprite sheets. Just React, fog of war, and Bresenham's line algorithm.*
