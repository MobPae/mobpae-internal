# Local Development Setup

Complete guide to getting all MobPae services running on your machine.

---

## Prerequisites

| Tool | Version | Install |
|---|---|---|
| Node.js | 20+ | https://nodejs.org |
| npm | 10+ | Comes with Node |
| PostgreSQL | 15+ | https://www.postgresql.org |
| Git | Any | https://git-scm.com |

---

## Repositories

Clone all five repos into the same parent folder:

```bash
mkdir mobpae && cd mobpae

git clone https://github.com/MobPae/mobpae-backend
git clone https://github.com/MobPae/mobpae-app
git clone https://github.com/MobPae/mobpae-employer
git clone https://github.com/MobPae/mobpae-admin
git clone https://github.com/MobPae/mobpae-website   # optional
```

---

## 1. Backend Setup

```bash
cd mobpae-backend
npm install
```

### Create `.env`

Copy the example and fill in values:

```bash
cp .env.example .env
```

Minimum required for local dev:

```env
PORT=3000
DATABASE_URL="postgresql://postgres:password@localhost:5432/mobpae"

JWT_SECRET=local-dev-secret-change-in-prod
JWT_EXPIRES_IN=7d

FRONTEND_URL=http://localhost:5175
CORS_ORIGINS=http://localhost:5173,http://localhost:5174,http://localhost:5175

ENABLE_SWAGGER=true

# Leave these as dummy values for local dev (file uploads won't work without real R2)
R2_ACCOUNT_ID=dummy
R2_ACCESS_KEY_ID=dummy
R2_SECRET_ACCESS_KEY=dummy
R2_BUCKET_NAME=mobpae-assets

# Leave these as dummy for local dev (Razorpay payments won't work)
RAZORPAY_KEY_ID=rzp_test_dummy
RAZORPAY_KEY_SECRET=dummy
RAZORPAY_WEBHOOK_SECRET=dummy

# Email (optional — skip for local dev, emails will fail silently)
SMTP_HOST=smtp-relay.brevo.com
SMTP_PORT=587
SMTP_USER=your-smtp-user
SMTP_PASS=your-smtp-password
MAIL_FROM=support@mobpae.com
```

### Create the database

```bash
# In psql or using createdb
createdb mobpae
```

### Run migrations + seed

```bash
npx prisma migrate dev
npx ts-node prisma/seed.ts
```

### Start the backend

```bash
npm run start:dev
```

Backend runs at: **http://localhost:3000**  
Swagger docs at: **http://localhost:3000/api**

---

## 2. Employee App Setup

```bash
cd mobpae-app
npm install
```

Create `.env.local`:
```env
VITE_API_URL=http://localhost:3000
```

```bash
npm run dev
```

Runs at: **http://localhost:5175**

---

## 3. Employer Portal Setup

```bash
cd mobpae-employer
npm install
```

Create `.env.local`:
```env
VITE_API_URL=http://localhost:3000
```

```bash
npm run dev
```

Runs at: **http://localhost:5174**

---

## 4. Admin Panel Setup

```bash
cd mobpae-admin
npm install
```

Create `.env.local`:
```env
VITE_API_URL=http://localhost:3000
```

```bash
npm run dev
```

Runs at: **http://localhost:5173**

---

## Port Map

| Service | URL |
|---|---|
| Backend API | http://localhost:3000 |
| Swagger UI | http://localhost:3000/api |
| Admin Panel | http://localhost:5173 |
| Employer Portal | http://localhost:5174 |
| Employee App | http://localhost:5175 |

---

## Startup Order

Always start in this order:

1. **PostgreSQL** (ensure it's running)
2. **Backend** (`npm run start:dev` in `mobpae-backend`)
3. **Any frontend** (order doesn't matter once backend is up)

---

## Verify Everything Works

1. Open http://localhost:3000/api — Swagger UI should load
2. Open http://localhost:5173 — Admin panel login should appear
3. Log in with: `admin@mobpae.com` / `Admin@1234`
4. Open http://localhost:5174 — Employer portal
5. Log in with: `employer@northstar.mobpae.com` / `Demo@1234`
6. Open http://localhost:5175 — Employee app
7. Log in with: `emp001@northstar.mobpae.com` / `Demo@1234`

See [test-accounts.md](./test-accounts.md) for all seeded accounts.

---

## Common Issues

**`ECONNREFUSED` on backend start**  
PostgreSQL is not running. Start it with `brew services start postgresql` (macOS) or `sudo service postgresql start` (Linux).

**`P1001: Can't reach database server`**  
Wrong `DATABASE_URL`. Check the host, port, username, password, and database name.

**`prisma migrate dev` asks for a name**  
Type any short description like `init` and press Enter.

**Frontend shows blank screen**  
Check that `VITE_API_URL` is set correctly and the backend is running.

**File uploads fail**  
R2 credentials are not configured. For local dev, KYC/selfie upload will fail — this is expected unless you set up a real R2 bucket.

---

## Reset Everything

To wipe the database and start fresh:

```bash
cd mobpae-backend
npx prisma migrate reset   # drops + recreates + runs migrations
npx ts-node prisma/seed.ts  # reseeds demo data
```

> ⚠️ Never run `prisma migrate reset` in production.
