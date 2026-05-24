# Endpoints & Data Contracts

JoyGemi does not run a traditional server-side API. All data is static, client-rendered, or delivered via third-party services (Firebase, Google Analytics). This document describes the data contracts: the shape of files served, URL patterns, and expected responses.

---

## Static Data Endpoints

These "endpoints" are static files served by the web server. No authentication is required.

---

### `GET /src/utils/games-data.js`

Returns the full game catalog as a JavaScript ES module.

**Response:** JavaScript module exporting `HTML5_GAMES_CATALOG`

**Content-Type:** `application/javascript`

**Schema (per entry):**

```ts
interface GameEntry {
  slug: string;           // URL-safe unique identifier
  id: string;             // Internal unique ID
  title: string;          // Display title
  img: string;            // Thumbnail URL or root-relative path
  embed: string;          // Iframe source URL
  isLocal?: boolean;      // True for /our_games/ hosted games
  fallbackUrl?: string;   // Backup embed URL
  category?: string;      // Explicit category key
  tags?: string[];        // Optional search tags
  disabled?: boolean;     // If true, game is hidden from catalog
}
```

---

### `GET /manifest.json`

Returns the Progressive Web App manifest.

**Content-Type:** `application/manifest+json`

**Key fields:**

```json
{
  "name": "JoyGemi",
  "short_name": "JoyGemi",
  "start_url": "/?v=8",
  "display": "standalone",
  "background_color": "#0f172a",
  "theme_color": "#0f172a",
  "icons": [...],
  "shortcuts": [
    { "name": "Popular Games", "url": "/#popular" },
    { "name": "New Games",     "url": "/#new" }
  ]
}
```

---

### `GET /sitemap.xml`

Returns the XML sitemap for search engine crawlers.

**Content-Type:** `application/xml`

The sitemap lists all indexable page URLs including the home page, about, FAQ, privacy policy, and terms. Game play pages (`/play/?id=...`) are included for top-level games.

---

### `GET /robots.txt`

```
User-agent: *
Disallow:

Sitemap: https://joygemi.com/sitemap.xml
```

All paths are open to crawling.

---

### `GET /ads.txt`

Declares authorised AdSense publishers per the IAB ads.txt specification.

```
google.com, pub-7608862327127348, DIRECT, f08c47fec0942fa0
```

---

### `GET /.well-known/assetlinks.json`

Declares the Android TWA package association for credential sharing and TWA validation.

---

## URL Patterns

### Home Page

```
GET /
```

Renders the full game catalog. Supports client-side filtering via tab clicks and search.

### Play Page

```
GET /play/?id={slug}
```

| Parameter | Required | Description |
|---|---|---|
| `id` | ✅ | The `slug` field of a game in `HTML5_GAMES_CATALOG`. Case-sensitive. |

**Valid example:**
```
/play/?id=shadow-fight-2
```

**Error state:** If `id` does not match any catalog entry, the play page renders an error card with a "Back to Home" link.

### Static Pages

| Path | Description |
|---|---|
| `/about/` | About JoyGemi |
| `/faq/` | Frequently Asked Questions |
| `/privacy-policy/` | Full Privacy Policy |
| `/terms-and-conditions/` | Terms of Service |
| `/privacy-center/` | User cookie/data controls |
| `/error.html` | Generic error page (linked from `.htaccess` ErrorDocument) |
| `/Maintenance/` | Maintenance splash (activated via `.htaccess` rewrite rule) |

---

## HTTP Headers

The `.htaccess` file sets the following response headers on all pages:

| Header | Value | Purpose |
|---|---|---|
| `X-Content-Type-Options` | `nosniff` | Prevent MIME-type sniffing |
| `X-Frame-Options` | `SAMEORIGIN` | Prevent clickjacking of the JoyGemi shell itself |
| `Referrer-Policy` | `strict-origin-when-cross-origin` | Control referrer leakage |
| `Permissions-Policy` | `geolocation=(), microphone=(), camera=()` | Deny unused APIs |

> `X-Frame-Options: SAMEORIGIN` applies to the JoyGemi **shell** pages. Game iframes (the partner embeds) are controlled by their own `X-Frame-Options` headers, not JoyGemi's.

---

## Third-Party Service Integrations

| Service | Purpose | Documentation |
|---|---|---|
| Firebase Auth (Google) | User sign-in | [auth.md](./auth.md) |
| Google Analytics (GA4) | Usage tracking | [tracking.md](../integration/tracking.md) |
| Google AdSense | Advertising | [adsense.google.com/start](https://adsense.google.com/start) |
| Google Tag Manager | Tag orchestration | [tagmanager.google.com](https://tagmanager.google.com) |

---

*Next: [SDK Usage →](../integration/sdk.md)*
