# JavaScript SDK

JoyGemi's JavaScript "SDK" is its set of ES modules — importable utilities that let you build features on top of the platform's game catalog, card renderer, and auth layer. This guide shows you how to use each module independently or compose them together.

---

## Installation

There is no npm package. Import the modules directly from the JoyGemi source. If you are building a feature inside the JoyGemi repo, use root-relative imports:

```js
import { HTML5_GAMES_CATALOG } from '/src/utils/games-data.js';
import { gameCardHTML, inferHtml5Category } from '/src/components/game-card.js';
import { auth, googleProvider } from '/auth_v4_joy.js';
import { getCookieConsent, setCookieConsent, getStoredTheme } from '/theme_v4_joy.js';
```

If you are consuming JoyGemi modules from an external project (e.g. a related subdomain), use the absolute URL:

```js
import { HTML5_GAMES_CATALOG } from 'https://joygemi.com/src/utils/games-data.js';
```

---

## Working with the Game Catalog

### Fetch All Games

```js
import { HTML5_GAMES_CATALOG } from '/src/utils/games-data.js';

console.log(`${HTML5_GAMES_CATALOG.length} games loaded`);
```

### Filter by Category

```js
import { HTML5_GAMES_CATALOG } from '/src/utils/games-data.js';
import { inferHtml5Category } from '/src/components/game-card.js';

function getGamesByCategory(categoryKey) {
  return HTML5_GAMES_CATALOG.filter((game) => {
    const cat = game.category ?? inferHtml5Category(game.title);
    return cat === categoryKey;
  });
}

const horrorGames = getGamesByCategory('horror');
```

### Search Games

```js
function searchGames(query) {
  const q = query.toLowerCase().trim();
  if (!q) return HTML5_GAMES_CATALOG;

  return HTML5_GAMES_CATALOG.filter((game) => {
    return (
      game.title.toLowerCase().includes(q) ||
      (game.tags ?? []).some((tag) => tag.includes(q)) ||
      (game.category ?? '').includes(q)
    );
  });
}

const results = searchGames('racing');
```

### Look Up a Game by Slug

```js
function getGameBySlug(slug) {
  return HTML5_GAMES_CATALOG.find((game) => game.slug === slug) ?? null;
}

const game = getGameBySlug('shadow-fight-2');
if (game) {
  console.log(game.title, game.embed);
}
```

---

## Rendering Game Cards

### Single Card

```js
import { gameCardHTML } from '/src/components/game-card.js';
import { HTML5_GAMES_CATALOG } from '/src/utils/games-data.js';

const game = HTML5_GAMES_CATALOG[0];
const cardHtml = gameCardHTML(game, 0);

document.getElementById('my-grid').insertAdjacentHTML('beforeend', cardHtml);
```

### Render a Grid of Cards

```js
import { gameCardHTML, generateSkeletonCards } from '/src/components/game-card.js';
import { HTML5_GAMES_CATALOG } from '/src/utils/games-data.js';

const grid = document.getElementById('game-grid');

// Show skeletons while "loading"
grid.innerHTML = generateSkeletonCards(12);

// Replace with real cards (simulated async pattern)
requestAnimationFrame(() => {
  grid.innerHTML = HTML5_GAMES_CATALOG
    .map((game, i) => gameCardHTML(game, i))
    .join('');
});
```

### Custom Card Template

If `gameCardHTML` doesn't suit your layout, build your own using the primitives:

```js
import {
  CAT_COLORS,
  CAT_ICONS,
  inferHtml5Category,
  buildGameCover
} from '/src/components/game-card.js';

function myCardHTML(game) {
  const category = game.category ?? inferHtml5Category(game.title);
  const color = CAT_COLORS[category] ?? '#6b7280';
  const icon = CAT_ICONS[category] ?? '🎲';
  const cover = buildGameCover(game);

  return `
    <div class="my-card" data-slug="${game.slug}" style="--accent: ${color}">
      <div class="cover">${cover}</div>
      <div class="info">
        <span class="cat">${icon} ${category}</span>
        <h3>${game.title}</h3>
      </div>
    </div>
  `;
}
```

---

## Authentication

### Sign In

```js
import { auth, googleProvider } from '/auth_v4_joy.js';

async function signIn() {
  const result = await auth.signInWithPopup(googleProvider);
  return result.user;
}
```

### Listen for Auth Changes

```js
import { auth } from '/auth_v4_joy.js';

auth.onAuthStateChanged((user) => {
  if (user) {
    console.log('Signed in:', user.displayName);
  } else {
    console.log('Signed out');
  }
});
```

See [Authentication](../api/auth.md) for the full auth reference.

---

## Theme and Consent

### Read / Write Cookie Consent

```js
import { getCookieConsent, setCookieConsent } from '/theme_v4_joy.js';

// Read
const consent = getCookieConsent(); // true | false | null

// Write (called when user clicks Accept or Decline in the consent banner)
setCookieConsent(true);   // accepted
setCookieConsent(false);  // declined
```

### Read Stored Theme

```js
import { getStoredTheme } from '/theme_v4_joy.js';

const theme = getStoredTheme(); // 'dark' | 'light'
document.body.setAttribute('data-theme', theme);
```

---

## Internationalisation

JoyGemi ships locale strings in `i18n_data.js`. Usage:

```js
import { i18nData } from '/i18n_data.js';

const locale = navigator.language.split('-')[0] ?? 'en';
const strings = i18nData[locale] ?? i18nData['en'];

console.log(strings.searchPlaceholder); // "Search games…"
```

---

## Version Pinning

All module imports in JoyGemi use cache-bust query strings:

```js
import { HTML5_GAMES_CATALOG } from './src/utils/games-data.js?v=20260520-persian-cover';
```

When you modify a module, update its version token in the import statement to force browsers to re-fetch the updated file.

---

*Next: [Embed Guide →](./embed.md)*
