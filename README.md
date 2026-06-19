# supabase-client-test

A minimal single-page HTML app to manually test Supabase authentication flows.

## What it tests

- Email + password sign in and sign up
- Forgot password / reset password via email link
- Google OAuth sign in
- Session persistence and JWT access token display
- Sign out

## Running

From the project root:

```bash
python3 -m http.server 3000
```

Then open [http://localhost:3000](http://localhost:3000).
