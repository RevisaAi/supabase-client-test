# CLAUDE.md — supabase-client-test

## Purpose

This is a zero-dependency, single-file HTML test harness for Supabase authentication. It exists so developers can manually verify that Supabase auth flows (email/password, OAuth, password reset) are working correctly against a real Supabase project before integrating them into a production app.

There is no build step, no framework, no package.json. Everything runs in the browser directly from static files.

## Project structure

```
index.html   — main app (all UI + JS in one file)
auth.html    — redirect shim: Supabase email links land here, hash is forwarded to index.html
README.md    — short user-facing instructions
CLAUDE.md    — this file
```

## Supabase project

- **URL:** `https://fvlmdvhjhktyescwkwzk.supabase.co`
- **Publishable key:** `sb_publishable_WcRGAEgIpoMkJ5hn5h5JzQ_smZuAW-R`

Both are hardcoded in `index.html` inside the `<script>` block at the top. They are publishable/anon keys — safe to commit.

## How to run

```bash
python3 -m http.server 3000
```

Open [http://localhost:3000](http://localhost:3000). No install required.

## What is currently implemented

| Feature | Status |
|---|---|
| Email + password sign in | Done |
| Email + password sign up | Done |
| Forgot password (send reset email) | Done |
| Reset password (set new password after email link) | Done |
| Google OAuth sign in | Done |
| Auth status badge (green/red) | Done |
| JWT access token display + copy button | Done |
| Sign out | Done |
| Session persistence on page reload | Done |

## How each flow works

### Email/password
Standard `signInWithPassword` / `signUp`. If sign up requires email confirmation, a message is shown instead of auto-login.

### Forgot password
1. User enters email → `resetPasswordForEmail` is called with `redirectTo: window.location.origin + window.location.pathname`
2. Supabase sends an email with a link pointing to `http://localhost:3000/auth`
3. `auth.html` receives the request and immediately redirects to `index.html` preserving the URL hash
4. `onAuthStateChange` fires with `PASSWORD_RECOVERY` event → reset password form is shown
5. User sets new password → `updateUser({ password })` is called

### Google OAuth
1. `signInWithOAuth({ provider: 'google', options: { redirectTo, queryParams: { prompt: 'select_account' } } })`
2. Browser navigates to Google → user picks account → Google redirects to Supabase callback
3. Supabase redirects back to `redirectTo` (the app URL) with session in the hash
4. `onAuthStateChange` fires with `SIGNED_IN` → session panel shown

## Supabase dashboard config required

These are one-time setup steps — already done for the current project:

- **Authentication → Providers → Google**: enabled with Google Cloud OAuth credentials
- **Authentication → URL Configuration → Redirect URLs**: `http://localhost:3000/index.html` is allowlisted
- **Google Cloud Console → OAuth Client → Authorized redirect URIs**: `https://fvlmdvhjhktyescwkwzk.supabase.co/auth/v1/callback`

If testing from a different port or domain, add it to both the Supabase redirect URLs allowlist and the Google OAuth client.

## How to extend

- All auth logic lives in the `<script>` block at the bottom of `index.html`
- UI panels are toggled via `showPanel(name)` — current panels: `auth`, `forgot`, `reset`, `session`
- To add a new auth provider (e.g. GitHub): add a button in `#auth-panel`, call `signInWithOAuth({ provider: 'github', ... })`, enable the provider in Supabase dashboard
- To add new post-login info (e.g. user metadata): extend the `setAuthenticated(session)` function and the `#session-panel` HTML

## Key Supabase JS patterns used

```js
// Init
const client = createClient(SUPABASE_URL, SUPABASE_KEY);

// Email sign in
client.auth.signInWithPassword({ email, password })

// OAuth
client.auth.signInWithOAuth({ provider: 'google', options: { redirectTo, queryParams: { prompt: 'select_account' } } })

// Password reset
client.auth.resetPasswordForEmail(email, { redirectTo })
client.auth.updateUser({ password: newPassword })

// Session
client.auth.getSession()
client.auth.onAuthStateChange((event, session) => { ... })
client.auth.signOut()
```

Supabase JS v2 is loaded from CDN: `https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2`
