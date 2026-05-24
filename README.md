# JoyGemi Developer Documentation

**JoyGemi** is a browser-based HTML5 game platform that lets players launch games instantly — no downloads, no signups, no email required. This documentation covers everything a developer, partner, or integrator needs to build on top of or alongside JoyGemi.

---

## What Is JoyGemi?

JoyGemi aggregates and hosts HTML5 games from first-party developers and third-party partners. Games are served in a clean, mobile-optimised interface with categories, search, favourites, and Google authentication. The platform is designed after Poki's distribution model: lightweight embeds, iframe-based game delivery, and real-time analytics.

- **Live site:** [https://joygemi.com](https://joygemi.com)
- **Game count:** 1,000+ HTML5 games
- **Primary market:** India (IN geo-targeted)
- **Tech stack:** Vanilla JS (ES modules), Firebase Auth, Google AdSense, Service Worker (PWA)

---

## Documentation Structure

```
docs/
├── intro.md                  Platform overview and architecture
├── getting-started.md        Set up your dev environment
├── games/
│   ├── overview.md           How the game catalog works
│   ├── categories.md         All supported game categories
│   └── how-games-work.md     Embed lifecycle, load flow, iframe API
├── api/
│   ├── reference.md          Full API reference index
│   ├── auth.md               Firebase Auth integration
│   └── endpoints.md          REST-style endpoints and data contracts
├── integration/
│   ├── sdk.md                JavaScript SDK usage
│   ├── embed.md              Embedding games via iframe
│   └── tracking.md           Analytics, events, and GTM
└── legal/
    ├── terms.md              Terms of Service
    └── privacy.md            Privacy Policy
```

---

## Quick Links

| I want to…                            | Go to                                      |
|---------------------------------------|--------------------------------------------|
| Understand the platform               | [Introduction](./docs/intro.md)            |
| Submit or list a game                 | [Games Overview](./docs/games/overview.md) |
| Embed a game on my site               | [Embed Guide](./docs/integration/embed.md) |
| Authenticate users via Google         | [Auth Docs](./docs/api/auth.md)            |
| Track game events                     | [Tracking](./docs/integration/tracking.md) |
| Review legal terms                    | [Terms](./docs/legal/terms.md)             |

---

## Contributing

This documentation lives alongside the JoyGemi source. To propose changes, open a pull request against the `docs/` directory. All Markdown files must pass linting (ATX headings, fenced code blocks, no bare URLs).

---

*© 2026 JoyGemi. All rights reserved.*
