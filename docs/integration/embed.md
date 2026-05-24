# Embedding Games

This guide covers two embedding scenarios:

1. **Embedding a JoyGemi game on an external site** — You want to display a JoyGemi-hosted or JoyGemi-listed game in your own web page.
2. **Adding a partner game to JoyGemi** — You are a game developer or partner wanting your game to appear on JoyGemi.

---

## 1. Embedding a JoyGemi Game on Your Site

JoyGemi games are served via iframe. You can embed any game by pointing an iframe at the game's `embed` URL.

### Basic Embed

```html
<iframe
  src="https://www.madkidgames.com/full/shadow-fight-2"
  width="800"
  height="600"
  allow="autoplay; fullscreen; gamepad"
  allowfullscreen
  title="Shadow Fight 2"
  loading="lazy"
  style="border: none;"
></iframe>
```

Replace the `src` with the `embed` URL from the game's entry in `HTML5_GAMES_CATALOG`.

### Responsive Full-Viewport Embed

For a play-page-style full-screen layout:

```html
<style>
  .game-wrapper {
    position: relative;
    width: 100%;
    padding-top: 56.25%; /* 16:9 ratio */
    overflow: hidden;
  }
  .game-wrapper iframe {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    border: none;
  }
</style>

<div class="game-wrapper">
  <iframe
    src="https://www.madkidgames.com/full/shadow-fight-2"
    allow="autoplay; fullscreen; gamepad; accelerometer; gyroscope"
    allowfullscreen
    title="Shadow Fight 2"
    loading="lazy"
  ></iframe>
</div>
```

### Embedding a First-Party JoyGemi Game

First-party games under `/our_games/` are hosted on `joygemi.com` and can be embedded freely:

```html
<iframe
  src="https://joygemi.com/our_games/wbwwb-v2/"
  width="960"
  height="640"
  allow="autoplay; fullscreen"
  allowfullscreen
  title="We Become What We Behold"
></iframe>
```

---

## 2. Using the JoyGemi Play Page as an Embed

Instead of embedding the raw game URL, you can embed the JoyGemi play page itself. This wraps the game in JoyGemi's UI chrome (title bar, fullscreen button, back link):

```html
<iframe
  src="https://joygemi.com/play/?id=shadow-fight-2"
  width="800"
  height="620"
  allow="autoplay; fullscreen; gamepad"
  allowfullscreen
  title="Shadow Fight 2 on JoyGemi"
></iframe>
```

> **Note:** Many browsers block embedding pages that set `X-Frame-Options: SAMEORIGIN`. If JoyGemi's shell pages block your embed, use the raw `embed` URL from the catalog entry instead.

---

## 3. Adding Your Game to JoyGemi (Partner Submission)

To have your HTML5 game listed on JoyGemi:

### Requirements

Your game must:

- Run entirely in a browser — no downloads, no plugins.
- Load within an `<iframe>` without setting `X-Frame-Options: DENY` on the game URL.
- Work on mobile devices (touch controls or adaptive input detection).
- Load within 5 seconds on a standard 4G connection.
- Not require the player to create an account before playing.
- Be free to play (monetisation via in-game ads is acceptable if declared).
- Not contain adult content, graphic violence directed at real people, or content illegal in India.

### Submission Format

Create a game entry object conforming to the [Game Entry Schema](../games/overview.md#game-entry-schema) and submit it via pull request to `src/utils/games-data.js`:

```js
{
  "slug": "your-game-slug",           // lowercase, hyphens, unique
  "id": "your-game-slug-yoursite",    // "{slug}-{your-domain-handle}"
  "title": "Your Game Title",
  "img": "https://yourdomain.com/games/your-game/thumb.jpg",  // 300×225 px recommended
  "embed": "https://yourdomain.com/games/your-game/play",
  "category": "action",               // see categories.md for valid keys
  "tags": ["optional", "search", "tags"]
}
```

### Thumbnail Requirements

| Property | Requirement |
|---|---|
| Format | JPEG or WebP preferred; PNG accepted |
| Dimensions | 300 × 225 px (4:3 ratio) |
| File size | Under 100 KB |
| Content | Game screenshot or key art; no letterboxing |
| Hosting | Self-hosted on a reliable CDN; JoyGemi does not mirror thumbnails |

### Iframe Compatibility Check

Before submitting, verify your game iframe loads correctly:

```html
<!-- Test page -->
<!DOCTYPE html>
<html>
<body>
  <iframe
    src="YOUR_EMBED_URL"
    width="800" height="600"
    allow="autoplay; fullscreen"
  ></iframe>
</body>
</html>
```

Open this in Chrome DevTools. If the Console shows:

```
Refused to display 'YOUR_EMBED_URL' in a frame because it set 'X-Frame-Options' to 'deny'.
```

Then your game server must remove or change that header to `SAMEORIGIN` or omit it entirely.

---

## 4. Hosting a Game on JoyGemi Directly

For first-party games fully hosted on `joygemi.com`:

1. Create a directory under `/our_games/{your-game-id}/`.
2. Add your game files. The entry point must be `/our_games/{your-game-id}/index.html`.
3. Add the catalog entry with `"isLocal": true`.
4. Test locally by serving the project and navigating to `/play/?id={your-slug}`.

Your game directory must be entirely self-contained. Do not rely on CDN-hosted scripts that may be blocked by CSP. Bundle all dependencies.

---

## iframe `allow` Attribute Reference

| Permission | When to include |
|---|---|
| `autoplay` | Game plays audio/video on load |
| `fullscreen` | Game uses the Fullscreen API |
| `gamepad` | Game supports gamepad/controller input |
| `accelerometer` | Mobile tilt-based input |
| `gyroscope` | Mobile orientation-based input |
| `clipboard-write` | Game writes to clipboard (e.g. share score) |
| `pointer-lock` | First-person camera control |

---

*Next: [Analytics & Tracking →](./tracking.md)*
