# Login Testing Guide for Chefcito

## Prerequisites

You need to complete the MongoDB Atlas setup to test login functionality.

### Step 1: Complete MongoDB Atlas Setup (if not already done)

1. Visit https://www.mongodb.com/cloud/atlas
2. Create a free-tier cluster (M0)
3. Create a database user with username `chefcito-app`
4. Allow network access from anywhere (0.0.0.0/0)
5. Get your connection string (it will look like):
   ```
   mongodb+srv://chefcito-app:YOUR_PASSWORD@cluster0.xxxxx.mongodb.net/?retryWrites=true&w=majority
   ```

### Step 2: Update Environment Variables

Edit `.env.development.local` and replace the MongoDB connection string placeholder:

```bash
# Find this line:
MONGODB_URI=mongodb+srv://chefcito-app:YOUR_PASSWORD_HERE@cluster0.xxxxx.mongodb.net/chefcito?retryWrites=true&w=majority

# Replace with your actual Atlas connection string:
MONGODB_URI=mongodb+srv://chefcito-app:YOUR_ACTUAL_PASSWORD@cluster0.xxxxx.mongodb.net/chefcito?retryWrites=true&w=majority
```

### Step 3: Create Test User

After updating your connection string, create a test user:

```bash
cd /vercel/share/v0-project
pnpm node scripts/create-test-user.mjs
```

You should see:
```
[Test] Connecting to MongoDB Atlas...
[Test] Connected successfully!
[Test] ✓ Test user created successfully!
[Test] Email: test@example.com
[Test] Password: password123
```

### Step 4: Test Login Flow

1. Navigate to http://localhost:3000/login
2. Click the "Login" tab (should be selected by default)
3. Enter credentials:
   - **Email or Username:** `test@example.com`
   - **Password:** `password123`
4. Click **Login** button
5. You should be redirected to the dashboard

## Troubleshooting

### Connection Failed: "Database connection failed"
- Check your MONGODB_URI in `.env.development.local`
- Verify the password is correctly URL-encoded (e.g., special characters like `@` → `%40`)
- Ensure network access is enabled in MongoDB Atlas (Security → Network Access)

### User Not Found: "Invalid credentials"
- Run the test user creation script again
- Check that the script completed successfully

### Password Mismatch: "Invalid credentials"
- Ensure you're using `test@example.com` and `password123`
- Try recreating the user with the script

### "Google (not configured)"
- Google OAuth is not required for testing the email/password login
- You can test the basic email/password flow without configuring Google OAuth

## Testing the API Directly

You can also test the login endpoint directly:

```bash
curl -X POST http://localhost:3000/api/users/login \
  -H "Content-Type: application/json" \
  -d '{
    "identifier": "test@example.com",
    "password": "password123"
  }'
```

Expected response:
```json
{
  "user": {
    "id": "test-user-001",
    "name": "Test User",
    "email": "test@example.com",
    "role": "Owner",
    "status": "Off Shift",
    "createdAt": "2026-06-25T08:00:00.000Z",
    "updatedAt": "2026-06-25T08:00:00.000Z"
  },
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

## Login Flow Architecture

The login system has two paths:

### 1. Email/Password Login
- **Route:** `POST /api/users/login`
- **Auth:** Supports both email and username
- **Password:** Hashed with bcryptjs, compares using `bcryptjs.compare()`
- **Token:** JWT token generated valid for 24 hours
- **Model:** Uses `User.findOne()` with email/username

### 2. Google OAuth Login
- **Route:** `POST /api/auth/google`
- **Auth:** Google ID token verification
- **Auto-creation:** Creates new users on first sign-up
- **Restaurant:** Automatically creates a restaurant for Owner-role users
- **Token:** Same JWT token system as email/password

## Authentication Storage

Logged-in users are stored in localStorage:
- `auth-token` - JWT token for API requests
- `auth-user` - User object (name, email, role, etc.)

The auth state is managed by Zustand store at `/src/lib/stores/auth-store.ts`
