# API Reference

JoyGemi is a static-site platform — it does not expose a traditional server-side REST API. Instead, the "API" is composed of:

1. **Client-side JavaScript modules** — importable utilities for building on top of JoyGemi's catalog and components.
2. **Firebase Auth** — the authentication layer (Google Sign-In via Firebase SDK).
3. **Data contracts** — the schema of the game catalog and events emitted to Google Analytics.

This reference indexes all three layers. Jump to the section you need.

---

## Modules

### `src/utils/games-data.js`

Exports the full game catalog.

```js
import { HTML5_GAMES_CATALOG } from './src/utils/games-data.js';
```

| Export | Type | Description |
|---|---|---|
| `HTML5_GAMES_CATALOG` | `GameEntry[]` | Array of all game objects. See [Games Overview](../games/overview.md) for the full schema. |

---

### `src/components/game-card.js`

Exports rendering utilities and constants.

```js
import {
  CAT_COLORS,
  CAT_ICONS,
  escapeSvgText,
  buildGameCover,
  inferHtml5Category,
  buildPartnerCover,
  generateSkeletonCards,
  gameCardHTML
} from './src/components/game-card.js';
```

| Export | Type | Signature | Description |
|---|---|---|---|
| `CAT_COLORS` | `object` | `{ [category: string]: string }` | Maps category key to hex color string. |
| `CAT_ICONS` | `object` | `{ [category: string]: string }` | Maps category key to emoji icon. |
| `escapeSvgText` | `function` | `(str: string) => string` | Escapes special characters for safe SVG `<text>` insertion. |
| `inferHtml5Category` | `function` | `(title: string) => string` | Infers a category key from a game title string. |
| `buildGameCover` | `function` | `(entry: GameEntry) => string` | Returns an HTML string for the game card cover image/SVG placeholder. |
| `buildPartnerCover` | `function` | `(entry: GameEntry) => string` | Returns an HTML string for a partner-sourced cover with fallback. |
| `generateSkeletonCards` | `function` | `(count: number) => string` | Returns `count` skeleton loading card HTML strings joined together. |
| `gameCardHTML` | `function` | `(entry: GameEntry, index: number) => string` | Returns the full HTML string for a single game card. |

---

### `theme_v4_joy.js`

Exports theme and consent utilities.

```js
import {
  getCookieConsent,
  setCookieConsent,
  getStoredTheme
} from './theme_v4_joy.js';
```

| Export | Signature | Description |
|---|---|---|
| `getCookieConsent` | `() => boolean \| null` | Returns the user's stored cookie consent preference (`true` = accepted, `false` = declined, `null` = not yet set). |
| `setCookieConsent` | `(accepted: boolean) => void` | Writes the user's cookie consent decision to `localStorage`. |
| `getStoredTheme` | `() => 'dark' \| 'light'` | Returns the user's stored theme preference, defaulting to `'dark'`. |

---

### `auth_v4_joy.js`

Initialises Firebase and exposes auth globals.

```js
import { app, auth, googleProvider } from './auth_v4_joy.js';
```

| Export | Type | Description |
|---|---|---|
| `app` | `FirebaseApp` | Initialised Firebase app instance. |
| `auth` | `Auth` | Firebase Auth service instance. Also exposed as `window.auth`. |
| `googleProvider` | `GoogleAuthProvider` | Pre-configured Google OAuth provider. Also available as `window.googleProvider`. |

See [Authentication](./auth.md) for full usage documentation.

---

## Global State

`main_v4_joy.js` maintains application state on `window` so that modules loaded in separate `<script type="module">` tags can share state:

| `window` Key | Type | Description |
|---|---|---|
| `window.ALL_GAMES` | `GameEntry[]` | Populated after catalog load. Full list of active games. |
| `window.activeTab` | `string` | Currently selected tab key (e.g. `'all'`, `'action'`). |
| `window.searchQ` | `string` | Current search query string. |
| `window.currentUser` | `FirebaseUser \| null` | Currently signed-in Firebase user, or `null`. |
| `window.FAVOURITES` | `Set<string>` | Set of slugs the user has favourited. Persisted to `localStorage`. |
| `window.RECENTLY_PLAYED` | `string[]` | Array of slugs played in the current session (most recent first). |
| `window.CAT_ICONS` | `object` | Reference to `CAT_ICONS` from `game-card.js`. |
| `window.generateSkeletonCards` | `function` | Reference to the skeleton card generator (used by `theme_v4_joy.js`). |

---

## Analytics Events

JoyGemi fires custom events to Google Analytics (GA4) for key user actions. See [Tracking](../integration/tracking.md) for the full event schema.

---

## Further Reading

- [Authentication →](./auth.md)
- [Endpoints & Data Contracts →](./endpoints.md)
- [SDK Usage →](../integration/sdk.md)
