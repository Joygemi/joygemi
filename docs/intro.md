# Introduction to JoyGemi

JoyGemi is an HTML5 browser game platform built for instant, frictionless play. This document explains what JoyGemi is, how it is architected, and where each piece of this documentation applies.

---

## Platform Overview

JoyGemi operates as a **game aggregator and publisher**. It curates HTML5 games from first-party studios and third-party partners, then serves them through a single, fast-loading web interface at [joygemi.com](https://joygemi.com).

Key design principles:

- **Zero friction** — No account, no download, no email gate before playing.
- **Mobile-first** — Every layout decision targets small screens first; desktop scales up.
- **PWA-capable** — JoyGemi registers a service worker and ships a `manifest.json`, making it installable on Android and iOS home screens.
- **Partner-friendly** — Games are embedded via iframe with a clean separation between platform shell and game content.

---

## Architecture at a Glance

```
┌─────────────────────────────────────────────────────┐
│                  joygemi.com (PWA Shell)             │
│                                                     │
│  main_v4_joy.js        theme_v4_joy.js              │
│  game-card.js          auth_v4_joy.js               │
│  games-data.js         i18n_data.js                 │
│                                                     │
│  ┌──────────┐   ┌──────────┐   ┌──────────────────┐ │
│  │ Home     │   │ Play     │   │ Legal / FAQ /     │ │
│  │ (index)  │   │ /play/   │   │ About pages       │ │
│  └──────────┘   └──────────┘   └──────────────────┘ │
│        │               │                            │
│        ▼               ▼                            │
│  Game Catalog     iframe Embed                      │
│  (games-data.js)  (partner URL or /our_games/)      │
└─────────────────────────────────────────────────────┘
         │
         ▼
┌─────────────────────┐     ┌─────────────────────────┐
│  Firebase Auth       │     │  Google AdSense          │
│  (Google sign-in)    │     │  (pub-7608862327127348)  │
└─────────────────────┘     └─────────────────────────┘
         │
         ▼
┌─────────────────────┐
│  Google Analytics    │
│  (G-VTTVP0KHN7)      │
└─────────────────────┘
```

---

## Core Modules

| File | Purpose |
|---|---|
| `main_v4_joy.js` | App bootstrap, game rendering, search, tabs, contact form |
| `theme_v4_joy.js` | Dark/light mode, cookie consent, auth UI state |
| `auth_v4_joy.js` | Firebase app init, Google OAuth provider, `window.auth` |
| `src/utils/games-data.js` | Static catalog of all games (slug, id, img, embed URL) |
| `src/components/game-card.js` | Card HTML builder, category inference, SVG cover generator |
| `i18n_data.js` | Internationalisation strings (English + additional locales) |
| `play/index.html` | Single-game play page — receives `?id=` param and renders iframe |
| `sw.js` | Service worker for offline support and asset caching |

---

## Platform URLs

| Path | Description |
|---|---|
| `/` | Home — full game catalog with search and tabs |
| `/play/?id={slug}` | Individual game play page |
| `/about/` | Platform About page |
| `/faq/` | Frequently Asked Questions |
| `/privacy-policy/` | Privacy Policy |
| `/terms-and-conditions/` | Terms and Conditions |
| `/privacy-center/` | User privacy controls |
| `/our_games/{id}/` | First-party hosted game root |

---

## Who This Documentation Is For

- **Game developers** wanting to publish a game on JoyGemi
- **Platform engineers** maintaining or extending the JoyGemi codebase
- **Integration partners** embedding JoyGemi games or using the tracking API
- **Legal / compliance** reviewers checking data practices

---

*Next: [Getting Started →](./getting-started.md)*
