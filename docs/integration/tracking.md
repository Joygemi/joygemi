# Analytics & Tracking

JoyGemi uses Google Analytics 4 (GA4) via Google Tag Manager to measure user behaviour, game engagement, and platform health. This document describes the tracking setup, all custom events, and how to extend tracking for new features.

---

## Setup

### Google Analytics 4

The GA4 measurement ID is `G-VTTVP0KHN7`. The tracking snippet is loaded in every page's `<head>`:

```html
<!-- Google tag (gtag.js) -->
<script async src="https://www.googletagmanager.com/gtag/js?id=G-VTTVP0KHN7"></script>
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){ dataLayer.push(arguments); }
  gtag('js', new Date());
  gtag('config', 'G-VTTVP0KHN7');
</script>
```

This auto-tracks page views, scroll depth, and outbound link clicks via GA4's enhanced measurement.

### Google AdSense

JoyGemi serves ads through Google AdSense (`pub-7608862327127348`). AdSense impressions, clicks, and revenue are tracked separately in Google AdSense reporting and are not surfaced in GA4.

### Cookie Consent Gate

Analytics and advertising tracking are gated on the user's cookie consent decision. Until the user accepts the consent banner, `gtag` calls should not fire personalised ad tracking. The consent state is stored in `localStorage` via `setCookieConsent()`.

For GDPR/consent mode compliance, update the GA4 config to use **Consent Mode v2**:

```js
// Before gtag('config', ...) — set default consent state
gtag('consent', 'default', {
  analytics_storage: 'denied',
  ad_storage: 'denied',
  wait_for_update: 500
});

// After user accepts — update consent
function onConsentAccepted() {
  setCookieConsent(true);
  gtag('consent', 'update', {
    analytics_storage: 'granted',
    ad_storage: 'granted'
  });
}
```

---

## Custom Events

JoyGemi fires the following custom GA4 events. All events use `gtag('event', ...)`.

### `game_launch`

Fired when a player navigates to the play page for a game.

```js
gtag('event', 'game_launch', {
  game_slug:     'shadow-fight-2',
  game_title:    'Shadow Fight 2',
  game_category: 'fighting',
  source:        'home_grid'   // 'home_grid' | 'search' | 'recently_played' | 'favourites'
});
```

| Parameter | Type | Description |
|---|---|---|
| `game_slug` | `string` | The game's `slug` field. |
| `game_title` | `string` | The game's display title. |
| `game_category` | `string` | The inferred or explicit category. |
| `source` | `string` | Where the user launched the game from. |

---

### `game_iframe_loaded`

Fired when the iframe `onload` event fires on the play page (game is visible and running).

```js
gtag('event', 'game_iframe_loaded', {
  game_slug:    'shadow-fight-2',
  load_time_ms: 1840   // time from navigation to iframe load in milliseconds
});
```

---

### `game_iframe_error`

Fired when an iframe fails to load.

```js
gtag('event', 'game_iframe_error', {
  game_slug:   'shadow-fight-2',
  embed_url:   'https://www.madkidgames.com/full/shadow-fight-2',
  fallback_attempted: true
});
```

---

### `favourite_added`

Fired when a user adds a game to their favourites.

```js
gtag('event', 'favourite_added', {
  game_slug:  'shadow-fight-2',
  user_uid:   'firebase_uid_here'   // only if user is signed in
});
```

---

### `favourite_removed`

Fired when a user removes a game from their favourites.

```js
gtag('event', 'favourite_removed', {
  game_slug: 'shadow-fight-2'
});
```

---

### `search_performed`

Fired when the user submits a search query (on input debounce).

```js
gtag('event', 'search_performed', {
  search_term:     'shooting games',
  results_count:   42
});
```

---

### `tab_switched`

Fired when the user clicks a category tab on the home page.

```js
gtag('event', 'tab_switched', {
  tab_key:   'action',
  tab_label: 'Action'
});
```

---

### `sign_in`

Fired when a user successfully completes Google Sign-In.

```js
gtag('event', 'sign_in', {
  method: 'google'
});
```

---

### `sign_out`

Fired when a user signs out.

```js
gtag('event', 'sign_out', {});
```

---

### `contact_form_submitted`

Fired when the contact form is submitted (before server response).

```js
gtag('event', 'contact_form_submitted', {
  form_id: 'home_contact'
});
```

---

## Firing Custom Events

To fire a custom event anywhere in the codebase:

```js
// Ensure gtag is available (it is loaded globally in <head>)
if (typeof gtag === 'function') {
  gtag('event', 'your_event_name', {
    param_one: 'value',
    param_two: 42
  });
}
```

Always guard with `typeof gtag === 'function'` — if the analytics script is blocked by an ad blocker, the global `gtag` will be undefined.

---

## Adding New Events

1. Define the event name (use `snake_case`).
2. Define all parameters the event should carry (use `snake_case` keys).
3. Add the `gtag('event', ...)` call in the relevant code path.
4. Add the event to this document (name, parameters, trigger condition).
5. Register the event and parameters as custom definitions in the GA4 property (Admin → Custom definitions) to ensure they appear in reports.

---

## Key Reports in GA4

| Report | Where to find | What it measures |
|---|---|---|
| Real-time | Reports → Realtime | Active users and events right now |
| Game launches | Explore → Freeform | `game_launch` events, broken down by `game_title` |
| Load failures | Explore → Freeform | `game_iframe_error` count and top failing slugs |
| Search terms | Reports → Engagement → Events → `search_performed` | Top search queries and result counts |
| Favourites | Events → `favourite_added` | Most-favourited games |
| User acquisition | Reports → Acquisition | Traffic sources, organic vs paid |

---

*Next: [Terms of Service →](../legal/terms.md)*
