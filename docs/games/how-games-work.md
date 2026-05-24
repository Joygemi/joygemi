# How Games Work

This document explains the full lifecycle of a game on JoyGemi — from catalog entry to player interaction — including the iframe architecture, the play page, first-party game hosting, and content security constraints.

---

## The Two-Page Model

JoyGemi uses a two-page model to separate catalog browsing from gameplay:

1. **Home page (`/`)** — Shows the catalog. Clicking a game card navigates to the play page.
2. **Play page (`/play/?id={slug}`)** — Loads a single game in an iframe, full-viewport.

This separation keeps the home page fast (no iframes pre-loaded) and gives each game its own canonical URL for SEO and sharing.

---

## Game Card → Play Page Navigation

When a player clicks a game card on the home page, the card's `onclick` handler calls `launchGame(slug)`:

```js
// main_v4_joy.js
function launchGame(slug) {
  window.location.href = `/play/?id=${encodeURIComponent(slug)}`;
}
```

The play page (`/play/index.html`) reads the `id` query parameter, looks up the matching entry in `HTML5_GAMES_CATALOG`, and renders the game iframe.

---

## The Play Page (`/play/`)

### URL Structure

```
https://joygemi.com/play/?id=shadow-fight-2
```

The `id` parameter must match the `slug` field of a catalog entry exactly.

### Iframe Loading Sequence

```
1. Player navigates to /play/?id={slug}
2. play_v4_joy.js reads URLSearchParams
3. Catalog lookup: find entry where entry.slug === id
4. If found:
   a. Set <title> to game title
   b. Update Open Graph meta tags for social sharing
   c. Set <iframe src> to entry.embed
   d. Show loading spinner
   e. iframe onload → hide spinner, show game
5. If not found:
   a. Show error state ("Game not found")
   b. Offer link back to home
```

### Iframe Attributes

JoyGemi applies these attributes to every game iframe:

```html
<iframe
  src="{embed_url}"
  allow="autoplay; fullscreen; gamepad; accelerometer; gyroscope"
  allowfullscreen
  loading="lazy"
  title="{game_title}"
></iframe>
```

For **local games** (`isLocal: true`), the `sandbox` attribute is omitted to allow full script execution. For **partner games**, a permissive sandbox is used:

```html
sandbox="allow-scripts allow-same-origin allow-popups allow-forms"
```

---

## Fallback URL Handling

If an embed URL becomes unavailable (CDN down, domain expired), entries with a `fallbackUrl` field are retried automatically:

```js
iframe.onerror = () => {
  if (entry.fallbackUrl) {
    iframe.src = entry.fallbackUrl;
  }
};
```

If neither URL works, the play page displays an error card with a retry button.

---

## First-Party Hosted Games

Games under `/our_games/` are served directly from JoyGemi's own domain. The current first-party title is **We Become What We Behold** (`/our_games/wbwwb-v2/`), a Phaser 3 narrative game.

### First-Party Game Directory Layout

```
our_games/wbwwb-v2/
├── index.html              # Phaser game entry point
├── css/
│   └── game.css
├── js/
│   ├── core/               # Scene and game loop base classes
│   ├── game/               # Director, Camera, entities
│   ├── peeps/              # Character behaviour classes
│   ├── scenes/             # Act I, II, III, credits
│   └── misc/               # Ambient effects (cricket, cursor, etc.)
├── sprites/                # All game art assets
│   ├── peeps/
│   ├── credits/
│   └── quote/
└── Persian-CoverImage.png  # Catalog thumbnail
```

### Building a First-Party Game

First-party games must:

- Be self-contained within their `/our_games/{id}/` directory.
- Load without any external dependencies that could fail (bundle third-party libs).
- Expose a standard HTML entry point at `/our_games/{id}/index.html`.
- Work at any viewport size without requiring a specific resolution.
- Not include any scripts that call `window.parent` or attempt to escape the iframe context.

---

## Progressive Web App (PWA) Caching

The service worker (`sw.js`) uses a **network-first, cache-fallback** strategy for HTML and JS assets, and **cache-first** for images.

Games themselves are **not cached** by the service worker — they are always fetched fresh from the partner or local server. This prevents stale game versions from being served after updates.

---

## Content Security Policy

JoyGemi's `.htaccess` sets Content Security Policy headers. Key directives relevant to games:

| Directive | Value | Purpose |
|---|---|---|
| `frame-src` | `*` | Allows embedding any partner domain |
| `script-src` | `'self' *.googleapis.com *.gstatic.com *.googlesyndication.com unpkg.com` | Trusted script origins |
| `img-src` | `* data:` | Allows thumbnails from any domain |

If you add a new game from a new partner domain that requires special CSP permissions (e.g. uses `SharedArrayBuffer`, requires COOP/COEP), update `.htaccess` and document the exception here.

---

## Fullscreen API

The play page exposes a fullscreen button. It calls the standard Fullscreen API on the iframe element:

```js
document.getElementById('gameIframe').requestFullscreen();
```

Fullscreen is supported in all modern browsers. On iOS Safari, `requestFullscreen()` is not available — instead, JoyGemi expands the iframe to `100dvh` with CSS as a visual fallback.

---

*Next: [API Reference →](../api/reference.md)*
