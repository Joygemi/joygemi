# Game Categories

JoyGemi organises its catalog into categories that power the tab navigation, search filtering, and card badge display on the home page. This document lists all supported categories, their display labels, associated icons, and the keyword rules used to auto-infer them.

---

## Category Reference Table

| Category Key | Display Label | Icon | Auto-Infer Keywords |
|---|---|---|---|
| `action` | Action | 🎮 | action, fight, battle, shoot, war, combat, blast |
| `adventure` | Adventure | 🗺️ | adventure, quest, explore, journey, world, saga |
| `arcade` | Arcade | 🕹️ | arcade, classic, retro, pac, breakout, pinball |
| `puzzle` | Puzzle | 🧩 | puzzle, match, block, sokoban, mahjong, brain, logic, word |
| `racing` | Racing | 🚗 | racing, race, drift, speed, car, truck, kart, moto, bike, driver |
| `sports` | Sports | ⚽ | soccer, football, basketball, baseball, tennis, golf, cricket, sport |
| `shooting` | Shooting | 🔫 | shoot, sniper, gun, bullet, fps, aim, target, crossbow |
| `fighting` | Fighting | 🥊 | fight, ninja, kung, boxer, warrior, brawl, pvp, stick |
| `horror` | Horror | 👻 | horror, scary, granny, escape, creep, evil, nun, monster, meat |
| `simulation` | Simulation | 🏗️ | simulator, manage, tycoon, idle, farm, city, build, craft |
| `strategy` | Strategy | ♟️ | strategy, chess, tower, defense, war, empire, kingdom |
| `multiplayer` | Multiplayer | 👥 | multiplayer, 2 player, versus, online, co-op |
| `casual` | Casual | 🌟 | casual, cute, jump, run, click, tap, flappy |
| `other` | Other | 🎲 | *(fallback — used when no keywords match)* |

---

## How Category Inference Works

Category inference is performed by `inferHtml5Category(title)` in `src/components/game-card.js`. It converts the game title to lowercase and checks for keyword presence in priority order (the table above is ordered by match priority — `action` is checked before `casual`).

```js
export function inferHtml5Category(title) {
  const t = title.toLowerCase();
  if (/action|fight|battle|shoot|war|combat|blast/.test(t)) return 'action';
  if (/adventure|quest|explore|journey|world|saga/.test(t))  return 'adventure';
  // ... and so on
  return 'other';
}
```

Because inference is imperfect, **always supply an explicit `category` field** in the game catalog entry for accurate categorisation. See [Games Overview → Game Entry Schema](./overview.md#game-entry-schema).

---

## Category Colors

Each category has a consistent accent color used on card badges. Colors are defined in `CAT_COLORS` in `game-card.js`:

| Category | Hex Color |
|---|---|
| `action` | `#ef4444` |
| `adventure` | `#f97316` |
| `arcade` | `#eab308` |
| `puzzle` | `#3b82f6` |
| `racing` | `#06b6d4` |
| `sports` | `#22c55e` |
| `shooting` | `#f43f5e` |
| `fighting` | `#a855f7` |
| `horror` | `#6b7280` |
| `simulation` | `#14b8a6` |
| `strategy` | `#8b5cf6` |
| `multiplayer` | `#ec4899` |
| `casual` | `#84cc16` |
| `other` | `#6b7280` |

---

## Tab Filtering

The home page exposes a tab bar that filters the visible card grid. Default tabs and their mapped categories:

| Tab Label | Filters To |
|---|---|
| All | *(all categories)* |
| Action | `action`, `fighting`, `shooting` |
| Puzzle | `puzzle` |
| Racing | `racing` |
| Sports | `sports` |
| Adventure | `adventure` |
| Horror | `horror` |
| Casual | `casual`, `arcade` |

Tab configuration is defined in `main_v4_joy.js`. To add a new tab, append an entry to the `TABS` array and map it to one or more category keys.

---

## Adding a New Category

1. Add the key, label, and icon to `CAT_COLORS` and `CAT_ICONS` in `game-card.js`.
2. Add the inference keywords to `inferHtml5Category()` in `game-card.js`.
3. Optionally add a tab for the new category in `main_v4_joy.js`.
4. Update this document with the new entry.

---

*Next: [How Games Work →](./how-games-work.md)*
