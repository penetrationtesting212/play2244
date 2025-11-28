# Login 400 Bad Request - Fix Guide

## Problem Identified

Your login request is returning `400 Bad Request` with `{"error":""}` which indicates a **database connection failure**.

## Root Cause

The backend cannot connect to PostgreSQL database, causing all authentication operations to fail silently.

## Diagnostic Results

✅ Backend server is running on port 3001  
✅ PostgreSQL service is running (postgresql-x64-17)  
✅ .env file exists with database configuration  
❌ **Backend cannot connect to database**

## Configuration Found

```env
DB_HOST=localhost
DB_PORT=5432
DB_NAME=playwrightcrx1
DB_USER=postgres
DB_PASSWORD=root
```

## Solution Steps

### Step 1: Verify PostgreSQL Connection

Open PowerShell and test the connection:

```powershell
# Install PostgreSQL client if not available
# Then test connection:
$env:PGPASSWORD="root"
psql -h localhost -p 5432 -U postgres -d postgres -c "\l"
```

If this fails, PostgreSQL might be running on a different port or the password is incorrect.

### Step 2: Find PostgreSQL Port

```powershell
# Check which port PostgreSQL is actually running on:
netstat -ano | findstr :5432
netstat -ano | findstr :5433

# Or check PostgreSQL config:
Get-Content "C:\Program Files\PostgreSQL\*\data\postgresql.conf" | Select-String "port"
```

### Step 3: Create Database (if needed)

```powershell
# Connect to PostgreSQL and create database:
$env:PGPASSWORD="root"
psql -h localhost -p 5432 -U postgres -d postgres

# Then run in psql:
CREATE DATABASE playwrightcrx1;
\c playwrightcrx1
```

### Step 4: Create User Table

```sql
-- Run this in psql or pgAdmin:
CREATE TABLE IF NOT EXISTS "User" (
  id VARCHAR(255) PRIMARY KEY,
  email VARCHAR(255) UNIQUE NOT NULL,
  password VARCHAR(255) NOT NULL,
  name VARCHAR(255) NOT NULL,
  "createdAt" TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  "updatedAt" TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE IF NOT EXISTS "RefreshToken" (
  id VARCHAR(255) PRIMARY KEY,
  token TEXT NOT NULL,
  "userId" VARCHAR(255) NOT NULL,
  "expiresAt" TIMESTAMP NOT NULL,
  "createdAt" TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  "revokedAt" TIMESTAMP,
  FOREIGN KEY ("userId") REFERENCES "User"(id) ON DELETE CASCADE
);
```

### Step 5: Create Test User

```sql
-- Run this in psql to create a test user:
-- Password is 'demo123' (bcrypt hashed)
INSERT INTO "User" (id, email, password, name, "createdAt", "updatedAt")
VALUES (
  gen_random_uuid()::text,
  'demo@example.com',
  '$2a$10$K7tWZqJ3nLxKQkL3PJ.kK.xZ8x8Z8Z8Z8Z8Z8Z8Z8Z8Z8Z8Z8Z8',
  'Demo User',
  NOW(),
  NOW()
)
ON CONFLICT (email) DO NOTHING;
```

**Note:** You'll need to generate the actual bcrypt hash. Use the backend to register a user instead.

### Step 6: Restart Backend with Proper Logging

```powershell
cd playwright-crx-enhanced\backend
npm start
```

Watch the console output for database connection messages.

## Alternative: Use Backend Migration Script

```powershell
cd playwright-crx-enhanced\backend

# Run migrations:
npm run migrate

# Or manually run:
node run-all-migrations.js
```

## Quick Test Commands

### Test 1: Check if database exists

```powershell
$env:PGPASSWORD="root"
psql -h localhost -p 5432 -U postgres -c "\l" | findstr playwrightcrx1
```

### Test 2: Check if tables exist

```powershell
$env:PGPASSWORD="root"
psql -h localhost -p 5432 -U postgres -d playwrightcrx1 -c "\dt"
```

### Test 3: Create a user via backend registration

```powershell
# Use PowerShell:
$body = @{
    email = "test@example.com"
    password = "Test123!"
    name = "Test User"
} | ConvertTo-Json

Invoke-WebRequest `
    -Uri "http://localhost:3001/api/auth/register" `
    -Method POST `
    -Body $body `
    -ContentType "application/json" `
    -UseBasicParsing
```

If registration works (returns 201), then try login:

```powershell
$body = @{
    email = "test@example.com"
    password = "Test123!"
} | ConvertTo-Json

Invoke-WebRequest `
    -Uri "http://localhost:3001/api/auth/login" `
    -Method POST `
    -Body $body `
    -ContentType "application/json" `
    -UseBasicParsing
```

## Common Issues

### Issue 1: Port Mismatch
- **Symptom:** Connection timeout or refused
- **Fix:** Check PostgreSQL is running on port 5432 (not 5433)
- **Command:** `netstat -ano | findstr :5432`

### Issue 2: Wrong Password
- **Symptom:** `password authentication failed`
- **Fix:** Verify password in .env matches PostgreSQL setup
- **Test:** `psql -h localhost -p 5432 -U postgres`

### Issue 3: Database Doesn't Exist
- **Symptom:** `database "playwrightcrx1" does not exist`
- **Fix:** Create the database:
  ```sql
  CREATE DATABASE playwrightcrx1;
  ```

### Issue 4: Tables Don't Exist
- **Symptom:** Backend connects but queries fail
- **Fix:** Run migrations or create tables manually

## Verification

After fixing, verify with these commands:

```powershell
# Test the diagnostic script:
cd C:\chandra-1212-main
node test-db-connection.js
```

Expected output:
```
✅ Backend is running
✅ Database connection is working!
✅ Login successful!
```

## Need More Help?

1. **Check backend logs:** Look at the terminal where backend is running
2. **Check PostgreSQL logs:** Usually in `C:\Program Files\PostgreSQL\17\data\log\`
3. **Enable debug logging:** Set `LOG_LEVEL=debug` in .env and restart backend

## Quick Fix Script

If you have Node.js and can access PostgreSQL, run this from the backend directory:

```javascript
// quick-fix.js
require('dotenv').config();
const { Pool } = require('pg');
const bcrypt = require('bcryptjs');
const { randomUUID } = require('crypto');

const pool = new Pool({
  connectionString: process.env.DATABASE_URL
});

async function quickFix() {
  const client = await pool.connect();
  
  // Create User table
  await client.query(`
    CREATE TABLE IF NOT EXISTS "User" (
      id VARCHAR(255) PRIMARY KEY,
      email VARCHAR(255) UNIQUE NOT NULL,
      password VARCHAR(255) NOT NULL,
      name VARCHAR(255) NOT NULL,
      "createdAt" TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
      "updatedAt" TIMESTAMP DEFAULT CURRENT_TIMESTAMP
    )
  `);
  
  // Create test user
  const hashedPassword = await bcrypt.hash('demo123', 10);
  await client.query(`
    INSERT INTO "User" (id, email, password, name, "createdAt", "updatedAt")
    VALUES ($1, $2, $3, $4, NOW(), NOW())
    ON CONFLICT (email) DO NOTHING
  `, [randomUUID(), 'demo@example.com', hashedPassword, 'Demo User']);
  
  client.release();
  await pool.end();
  console.log('✅ Database setup complete!');
}

quickFix().catch(console.error);
```

Save this as `quick-fix.js` in the backend directory and run:

```powershell
cd playwright-crx-enhanced\backend
node quick-fix.js
```

## Summary

The 400 error is caused by database connectivity issues. Follow these steps:

1. ✅ Verify PostgreSQL is running
2. ✅ Check DATABASE_URL in .env
3. ✅ Ensure database `playwrightcrx1` exists
4. ✅ Create User and RefreshToken tables
5. ✅ Create a test user
6. ✅ Restart backend and test login

Once the database is properly set up, the login endpoint should work correctly!
