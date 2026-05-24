# Getting Started

This guide walks you through setting up a local JoyGemi development environment, understanding the project layout, and making your first change.

---

## Prerequisites

| Requirement | Version | Notes |
|---|---|---|
| Modern browser | Chrome 90+, Firefox 88+, Safari 15+ | ES module support required |
| Node.js | 18 LTS or later | Only needed if running build/lint tools |
| Git | Any recent version | For cloning and version control |
| Firebase project | Any plan | Required for Auth; free Spark plan works |
| Google AdSense account | Optional | Only needed for ad monetisation |

> **No build step is required for local development.** JoyGemi uses native ES modules loaded directly by the browser. You do not need Webpack, Vite, or any bundler to run the project.

---

## 1. Clone the Repository

```bash
git clone https://github.com/your-org/joygemi.git
cd joygemi
```

---

## 2. Configure Firebase

JoyGemi uses Firebase for Google Sign-In. You must supply your own Firebase project credentials before running the app.

1. Go to [console.firebase.google.com](https://console.firebase.google.com) and create a project (or open an existing one).
2. Enable **Authentication → Sign-in method → Google**.
3. In **Project Settings → General → Your apps**, register a Web app and copy the config object.
4. Open `auth_v4_joy.js` and replace the `firebaseConfig` object:

```js
// auth_v4_joy.js
const firebaseConfig = {
  apiKey: "YOUR_API_KEY",
  authDomain: "YOUR_PROJECT.firebaseapp.com",
  projectId: "YOUR_PROJECT",
  storageBucket: "YOUR_PROJECT.firebasestorage.app",
  messagingSenderId: "YOUR_SENDER_ID",
  appId: "YOUR_APP_ID"
};
```

> **Never commit real API keys to a public repository.** For production, consider loading `firebaseConfig` from an environment-injected config endpoint or CI secret.

---

## 3. Serve Locally

Because JoyGemi uses ES modules and iframes, you must serve it over HTTP — opening `index.html` as a `file://` URL will not work.

**Option A — Python (no install required):**

```bash
python3 -m http.server 3000
```

**Option B — Node `serve` package:**

```bash
npx serve . -p 3000
```

**Option C — VS Code Live Server extension:**

Install the [Live Server extension](https://marketplace.visualstudio.com/items?itemName=ritwickdey.LiveServer), right-click `index.html` → **Open with Live Server**.

Then open `http://localhost:3000` in your browser.

---

## 4. Explore the Project Structure

```
joygemi/
├── index.html                   # Home page shell
├── main_v4_joy.js               # Core app logic
├── theme_v4_joy.js              # Theme + cookie consent
├── auth_v4_joy.js               # Firebase Auth setup
├── app_v1_joy.css               # Global styles
├── i18n_data.js                 # Locale strings
├── manifest.json                # PWA manifest
├── sw.js                        # Service worker
├── robots.txt                   # Crawl directives
├── sitemap.xml                  # XML sitemap
├── ads.txt                      # AdSense publisher declaration
│
├── src/
│   ├── utils/
│   │   └── games-data.js        # Full game catalog (slug, embed, img)
│   ├── components/
│   │   └── game-card.js         # Card renderer + category inference
│   └── scripts/
│       └── search-test.js       # Dev utility: test search ranking
│
├── play/
│   ├── index.html               # Single-game play page
│   ├── play_v4_joy.js           # Play page logic (iframe loader)
│   └── play_v4_joy.css          # Play page styles
│
├── our_games/
│   └── wbwwb-v2/               # First-party hosted game (Phaser)
│
├── about/                       # About page
├── faq/                         # FAQ page
├── privacy-policy/              # Privacy Policy
├── privacy-center/              # User privacy controls
├── terms-and-conditions/        # Terms of Service
├── Maintenance/                 # Maintenance splash
└── .well-known/
    └── assetlinks.json          # Android TWA asset linking
```

---

## 5. Adding a Game to the Catalog

Games are defined in `src/utils/games-data.js` as entries in the `HTML5_GAMES_CATALOG` array.

**Minimal entry:**

```js
{
  "slug": "my-awesome-game",
  "id": "my-awesome-game-joygemi",
  "img": "https://example.com/games/my-awesome-game/thumb.jpg",
  "title": "My Awesome Game",
  "embed": "https://example.com/games/my-awesome-game/play"
}
```

**First-party hosted game:**

```js
{
  "slug": "my-local-game",
  "id": "my-local-game-local",
  "img": "/our_games/my-local-game/cover.png",
  "title": "My Local Game",
  "embed": "/our_games/my-local-game/",
  "isLocal": true
}
```

See [Games Overview](./games/overview.md) for the full field reference.

---

## 6. Running the Search Test Script

JoyGemi ships a search ranking utility at `src/scripts/search-test.js`. Run it directly in a browser console or with Node:

```bash
node src/scripts/search-test.js
```

It prints match scores for test queries against the catalog, which helps verify that new slugs and titles surface correctly in the live search.

---

## 7. Deploying

JoyGemi is a **static site** — no server-side rendering, no backend API. Deploy by copying all files to any static host.

**Recommended hosts:**

- [Cloudflare Pages](https://pages.cloudflare.com) — zero config, global CDN, edge caching
- [Vercel](https://vercel.com) — simple drag-and-drop or Git integration
- [GitHub Pages](https://pages.github.com) — free for open-source repos
- Any standard Apache/Nginx web server (see `.htaccess` for existing rewrite rules)

The included `.htaccess` handles:
- Pretty-URL rewrites for `/play/` routes
- Caching headers for static assets
- HTTPS redirect enforcement
- Security headers (X-Frame-Options, CSP)

---

*Next: [Games Overview →](./games/overview.md)*
