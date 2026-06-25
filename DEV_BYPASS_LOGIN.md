# Development Login Bypass Guide

## Overview

In development mode, you can bypass the login screen and automatically access the application as a demo user. This is useful for testing the app without setting up a database or user accounts.

## Current Status

✅ **Login bypass is ENABLED** for development

Your dev user credentials:
- **Email:** dev@example.com
- **Name:** Dev User
- **Role:** Owner
- **Status:** On Shift

## How It Works

The auth provider (`src/components/layout/auth-provider.tsx`) automatically:

1. Checks if `NEXT_PUBLIC_DEV_BYPASS_LOGIN=true` is set in `.env.development.local`
2. Detects development mode (`process.env.NODE_ENV === 'development'`)
3. Creates a demo user and stores it in localStorage on app startup
4. Allows full app access without login credentials

## Configuration

The bypass is controlled by this environment variable in `.env.development.local`:

```env
NEXT_PUBLIC_DEV_BYPASS_LOGIN=true
```

### To Disable the Bypass

Set the variable to `false` or remove it:

```env
NEXT_PUBLIC_DEV_BYPASS_LOGIN=false
```

Then restart your dev server (`pnpm dev`).

## Usage

1. **Start the dev server:**
   ```bash
   pnpm dev
   ```

2. **Navigate to any page:**
   - `http://localhost:3000/pos` - Point of Sale system
   - `http://localhost:3000/kds` - Kitchen Display System
   - `http://localhost:3000/restaurant` - Restaurant settings
   - Any other protected route

3. **You're automatically authenticated** - No login required!

## Important Notes

- **Development Only:** This bypass only works when `NODE_ENV === 'development'`
- **Public Variable:** `NEXT_PUBLIC_` prefix means it's exposed to the browser (safe for dev mode)
- **Local Only:** Never commit production credentials with this bypass enabled
- **For Production:** Remove this logic before deploying (it won't run in production anyway)

## Console Output

When the bypass is active, you'll see this log message:

```
[Auth] Development mode: Auto-logged in as Dev User
```

## Testing Production Auth

To test the actual login flow in development:

1. Disable the bypass:
   ```env
   NEXT_PUBLIC_DEV_BYPASS_LOGIN=false
   ```

2. Restart the dev server

3. You'll be redirected to `/login` and must enter valid credentials

4. Create a test user in MongoDB or use the `/register` route

## Toggling Between Modes

You can quickly switch between dev bypass and production auth by:

```bash
# Development (auto-login)
echo "NEXT_PUBLIC_DEV_BYPASS_LOGIN=true" >> .env.development.local
pnpm dev

# Production-like (login required)
echo "NEXT_PUBLIC_DEV_BYPASS_LOGIN=false" >> .env.development.local
pnpm dev
```

## Troubleshooting

**Q: Still seeing login page after enabling bypass?**
- Make sure you've set `NEXT_PUBLIC_DEV_BYPASS_LOGIN=true` in `.env.development.local`
- Restart the dev server (`pnpm dev`)
- Clear browser cache/localStorage: press F12 → Application → Clear storage
- Refresh the page

**Q: Why does it redirect to /login after logout?**
- This is expected. The logout clears the user from the store, and the normal auth flow kicks in
- To re-enable dev bypass after logout, reload the page

**Q: Can I customize the dev user?**
- Yes! Edit `src/components/layout/auth-provider.tsx` lines 33-40 to change user properties
- Modify fields like `name`, `email`, `role`, or `status`
