# Games Overview

The JoyGemi game catalog is the heart of the platform. This document explains how games are structured, stored, and rendered, and what it means to "publish" a game on JoyGemi.

---

## Catalog Structure

All games are stored in a single static JavaScript file:

```
src/utils/games-data.js
```

This file exports a named array `HTML5_GAMES_CATALOG`. Each element represents one game. At build time (or at runtime, since there is no build step), `main_v4_joy.js` imports this array and constructs the UI.

---

## Game Entry Schema

Every game object supports the following fields:

| Field | Type | Required | Description |
|---|---|---|---|
| `slug` | `string` | ✅ | URL-safe identifier. Used in `/play/?id={slug}`. Must be unique and lowercase with hyphens. |
| `id` | `string` | ✅ | Internal unique ID. Convention: `{slug}-{source}` (e.g. `granny-madkidgames`). |
| `title` | `string` | ✅ | Display name shown on cards and in the play page `<title>`. |
| `img` | `string` | ✅ | Absolute URL or root-relative path to the game thumbnail (recommended: 300×225 px, JPEG or WebP). |
| `embed` | `string` | ✅ | URL loaded inside the game iframe. Can be an external partner URL or a local path under `/our_games/`. |
| `isLocal` | `boolean` | — | Set to `true` for games hosted under `/our_games/`. Affects CSP and iframe sandbox flags. |
| `fallbackUrl` | `string` | — | Alternative embed URL tried if the primary `embed` fails to load (used for games with CDN instability). |
| `category` | `string` | — | Explicit category string. If omitted, category is inferred from the title by `inferHtml5Category()` in `game-card.js`. |
| `tags` | `string[]` | — | Optional freeform tags for search ranking and filtering. |

**Full example:**

```js
{
  "slug": "shadow-fight-2",
  "id": "shadow-fight-2-madkidgames",
  "title": "Shadow Fight 2",
  "img": "https://www.madkidgames.com/games/shadow-fight-2/thumb_2.jpg",
  "embed": "https://www.madkidgames.com/full/shadow-fight-2",
  "category": "fighting",
  "tags": ["pvp", "ninja", "action"]
}
```

---

## How the Catalog Is Loaded

`main_v4_joy.js` imports `HTML5_GAMES_CATALOG` at the top of the module:

```js
import { HTML5_GAMES_CATALOG } from './src/utils/games-data.js?v=20260520';
```

The query string `?v=...` is a cache-bust token. Update it whenever you modify `games-data.js` to ensure users receive the latest catalog.

On page load, `buildHtml5Games()` maps each catalog entry to a game-card DOM element using `gameCardHTML()` from `game-card.js`. Cards are injected into the grid container in `index.html`.

---

## Category Inference

If a game entry does not specify a `category`, the platform infers it from the `title` string using keyword matching:

```js
// src/components/game-card.js
export function inferHtml5Category(title) { ... }
```

Categories currently supported by the inference engine are listed in [Categories](./categories.md). To guarantee correct categorisation, always supply an explicit `category` field.

---

## Game Sources

JoyGemi serves games from two sources:

### Partner Embeds

Most games are sourced from partner sites (e.g. `madkidgames.com`). The `embed` field contains the partner's full-screen game URL. JoyGemi loads it in a sandboxed iframe.

- JoyGemi does not host partner game assets.
- Partner availability affects whether the game loads.
- `fallbackUrl` can provide a backup embed if a partner URL becomes unavailable.

### First-Party Games

Games under `/our_games/` are hosted and maintained directly by JoyGemi. These are typically original titles built with Phaser or vanilla canvas. Set `"isLocal": true` on the catalog entry.

Current first-party games:

| Slug | Title | Engine |
|---|---|---|
| `we-become-what-we-behold` | We Become What We Behold | Phaser 3 |

---

## Adding a New Game

1. Add an entry to `HTML5_GAMES_CATALOG` in `src/utils/games-data.js`.
2. Update the cache-bust version in the import path in `main_v4_joy.js`.
3. Ensure the thumbnail URL returns a valid image (200 OK, correct MIME type).
4. Verify the embed URL loads in an iframe without `X-Frame-Options: DENY` blocking it.
5. Test locally at `http://localhost:3000`.

---

## Removing or Disabling a Game

To disable a game without deleting its entry, add a `"disabled": true` flag:

```js
{
  "slug": "old-game",
  "disabled": true,
  ...
}
```

The `buildHtml5Games()` function skips entries where `disabled === true`. This preserves the entry in the catalog for reference without rendering a card.

---

*Next: [Game Categories →](./categories.md)*
