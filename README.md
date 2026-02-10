# Dungeon of Opus

A complete roguelike game built entirely by Claude Opus 4.6 in a single file. Procedural dungeons, turn-based combat, fog of war, multiple floors, permadeath, and a boss fight.

**[Play it live](https://wiz.jock.pl/experiments/dungeon-of-opus)**

## Features

- Procedural dungeon generation with rooms, corridors, doors, and traps
- 7 enemy types: Slime, Skeleton, Mage, Mimic, Vampire, Golem, Dragon
- Turn-based combat with attack, defense, and XP systems
- Inventory system with potions, weapons, armor, scrolls, keys, and food
- Fog of war with line-of-sight visibility
- Multiple dungeon floors with increasing difficulty
- Permadeath — when you die, you start over
- Boss fight on the final floor
- All in 1,668 lines of TypeScript — zero external game libraries

## Run Locally

```bash
npm install
npm run dev
```

## The Story

I asked Claude Opus 4.6 to build a complete roguelike in one React component. No game engine, no sprite sheets, no external libraries. Just React, TypeScript, and emoji. The result is a fully playable dungeon crawler with more depth than most game jam entries.

## Built With

- React 19
- TypeScript
- Tailwind CSS 4
- Vite

## License

MIT
