# Authentication

JoyGemi uses **Firebase Authentication** with Google Sign-In as the sole identity provider. Authentication is optional — players can browse and play all games without signing in. Signing in unlocks favourites persistence and a personalised recently-played list.

---

## Firebase Setup

Authentication is initialised in `auth_v4_joy.js` using the Firebase Compat SDK (v10):

```html
<!-- Load Firebase Compat SDK before auth_v4_joy.js -->
<script src="https://www.gstatic.com/firebasejs/10.14.1/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.14.1/firebase-auth-compat.js"></script>
<script type="module" src="/auth_v4_joy.js"></script>
```

`auth_v4_joy.js` initialises the app and binds the auth instance to `window`:

```js
const app = firebase.initializeApp(firebaseConfig);
const auth = firebase.auth();
const googleProvider = new firebase.auth.GoogleAuthProvider();

window.auth = auth;
window.googleProvider = googleProvider;
```

> **Why `window.auth`?** `theme_v4_joy.js` and `main_v4_joy.js` are loaded as separate ES modules. Exporting via `window` is the simplest way to share the auth instance across modules without a module bundler.

---

## Signing In

Google Sign-In uses a popup flow:

```js
// Trigger Google Sign-In
async function signInWithGoogle() {
  try {
    const result = await window.auth.signInWithPopup(window.googleProvider);
    const user = result.user;
    console.log('Signed in as:', user.displayName, user.email);
  } catch (error) {
    console.error('Sign-in failed:', error.code, error.message);
  }
}
```

**Common error codes:**

| Code | Meaning | Suggested handling |
|---|---|---|
| `auth/popup-closed-by-user` | User dismissed the popup | Show a "Sign-in cancelled" toast; do nothing else |
| `auth/popup-blocked` | Browser blocked the popup | Show a message asking the user to allow popups |
| `auth/cancelled-popup-request` | Another popup was opened before this one resolved | Ignore; the newer popup supersedes this one |
| `auth/network-request-failed` | Network issue during auth | Show a retry option |

---

## Signing Out

```js
async function signOut() {
  await window.auth.signOut();
  window.currentUser = null;
  // Update UI: hide user avatar, clear FAVOURITES, etc.
}
```

---

## Observing Auth State

Use `onAuthStateChanged` to reactively update the UI whenever the auth state changes. This is the canonical way to know if a user is signed in — never rely on cached state:

```js
window.auth.onAuthStateChanged((user) => {
  window.currentUser = user;

  if (user) {
    // User is signed in
    renderUserAvatar(user.photoURL, user.displayName);
    loadFavourites(user.uid);
  } else {
    // User is signed out
    renderSignInButton();
    window.FAVOURITES = new Set();
  }
});
```

`onAuthStateChanged` fires once immediately on page load (with the current user or `null`), then again whenever state changes (sign-in, sign-out, token refresh).

---

## User Object

Firebase returns a `User` object on successful sign-in. Relevant properties:

| Property | Type | Description |
|---|---|---|
| `uid` | `string` | Unique Firebase user ID. Stable across sessions and devices. |
| `displayName` | `string \| null` | User's Google account display name. |
| `email` | `string \| null` | User's Google email address. |
| `photoURL` | `string \| null` | URL to user's Google profile photo. |
| `emailVerified` | `boolean` | Always `true` for Google-authenticated users. |

---

## Favourites Persistence

Favourites are stored in `localStorage` keyed by the user's `uid`:

```js
const FAVOURITES_KEY = `joygemi_favs_${user.uid}`;

function loadFavourites(uid) {
  const stored = localStorage.getItem(`joygemi_favs_${uid}`);
  window.FAVOURITES = stored ? new Set(JSON.parse(stored)) : new Set();
}

function saveFavourites(uid) {
  localStorage.setItem(
    `joygemi_favs_${uid}`,
    JSON.stringify([...window.FAVOURITES])
  );
}
```

If the user is not signed in, favourites still work but are stored under a generic key and lost on session end.

---

## Security Considerations

- Firebase API keys in `auth_v4_joy.js` are **client-visible by design** — this is standard Firebase practice. Access control is enforced by Firebase Security Rules and Authorized Domains, not by keeping the key secret.
- Add your production domain to **Firebase Console → Authentication → Settings → Authorized Domains** to prevent OAuth from running on unauthorised origins.
- Do not use Firebase API keys for any server-side privileged operations. Use a Firebase Admin SDK service account for server-side work.

---

## Android TWA Asset Linking

JoyGemi supports installation as a Trusted Web Activity (TWA) on Android. The `/.well-known/assetlinks.json` file declares the linked Android app package for passkey and credential-sharing purposes:

```json
[{
  "relation": ["delegate_permission/common.handle_all_urls"],
  "target": {
    "namespace": "android_app",
    "package_name": "com.joygemi.app",
    "sha256_cert_fingerprints": ["..."]
  }
}]
```

Update `sha256_cert_fingerprints` with your Android signing certificate fingerprint when building a native wrapper app.

---

*Next: [Endpoints & Data Contracts →](./endpoints.md)*
