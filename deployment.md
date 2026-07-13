# Deployment

How to deploy each MobPae service to production.

---

## Architecture

```
Internet → CDN (Cloudflare)
              ├── admin.mobpae.com    → Cloudflare Pages (mobpae-admin)
              ├── employer.mobpae.com → Cloudflare Pages (mobpae-employer)
              ├── app.mobpae.com      → Cloudflare Pages (mobpae-app)
              └── api.mobpae.com      → Backend server (Railway / EC2 / VPS)
                                           └── PostgreSQL (Supabase / RDS / Neon)
```

---

## Backend Deployment

### Environment Variables (Required in Production)

```env
# Server
PORT=3000
NODE_ENV=production

# Database
DATABASE_URL=postgresql://user:pass@host:5432/mobpae

# Auth — use a strong random secret (min 64 chars)
JWT_SECRET=<64-char-random-string>
JWT_EXPIRES_IN=7d

# CORS — list all frontend origins exactly
CORS_ORIGINS=https://admin.mobpae.com,https://employer.mobpae.com,https://app.mobpae.com
FRONTEND_URL=https://app.mobpae.com

# Swagger — disable in production
ENABLE_SWAGGER=false

# Cloudflare R2
R2_ACCOUNT_ID=<cloudflare-account-id>
R2_ACCESS_KEY_ID=<r2-access-key>
R2_SECRET_ACCESS_KEY=<r2-secret-key>
R2_BUCKET_NAME=mobpae-assets

# Razorpay — use live keys in production
RAZORPAY_KEY_ID=rzp_live_xxxx
RAZORPAY_KEY_SECRET=<razorpay-key-secret>
RAZORPAY_WEBHOOK_SECRET=<razorpay-webhook-secret>

# Email (Brevo / SendGrid / AWS SES)
SMTP_HOST=smtp-relay.brevo.com
SMTP_PORT=587
SMTP_USER=<smtp-user>
SMTP_PASS=<smtp-password>
SMTP_SECURE=false
MAIL_FROM=support@mobpae.com
MAIL_FROM_NAME=MobPae
```

### Build and Start

```bash
npm run build
npm run start:prod
```

### Database Migrations (Production)

**Never use `prisma migrate reset` in production.**

Apply pending migrations only:
```bash
npx prisma migrate deploy
```

Run this before starting the server when deploying schema changes.

### Recommended Hosting: Railway

1. Connect GitHub repo → Railway auto-deploys on push to `main`
2. Set all environment variables in Railway dashboard
3. Add a PostgreSQL service in Railway (or use external Supabase/Neon)
4. Set start command: `npm run start:prod`
5. Set build command: `npm run build && npx prisma migrate deploy`
6. Configure custom domain: `api.mobpae.com`

---

## Frontend Deployment (Cloudflare Pages)

All three frontends are static Vite builds deployed to Cloudflare Pages.

### Build Settings (same for all three)

| Setting | Value |
|---|---|
| Build command | `npm run build` |
| Output directory | `dist` |
| Node version | 20 |

### Environment Variables Per Frontend

**Admin Panel (`admin.mobpae.com`):**
```env
VITE_API_URL=https://api.mobpae.com
```

**Employer Portal (`employer.mobpae.com`):**
```env
VITE_API_URL=https://api.mobpae.com
```

**Employee App (`app.mobpae.com`):**
```env
VITE_API_URL=https://api.mobpae.com
VITE_RAZORPAY_KEY_ID=rzp_live_xxxx
```

### Deploy Steps

1. Connect GitHub repo to Cloudflare Pages
2. Set build settings and environment variables
3. Set custom domain
4. Every push to `main` triggers automatic redeploy

---

## Razorpay Webhook Setup (Production)

In Razorpay Dashboard → Settings → Webhooks → Add Webhook:

- **URL:** `https://api.mobpae.com/webhooks/razorpay`
- **Secret:** set to `RAZORPAY_WEBHOOK_SECRET`
- **Events to subscribe:**
  - `payment.captured`
  - `payment.failed`
  - `order.paid`

---

## Cloudflare R2 Setup

1. Create a bucket named `mobpae-assets` (or your `R2_BUCKET_NAME` value)
2. Set bucket to **Private** (no public access)
3. Create an R2 API token: R2 → Manage R2 API Tokens → Create Token
4. Give it Read + Write permissions for your bucket
5. Copy `Access Key ID` and `Secret Access Key` to backend env

---

## Production Checklist

Before going live:

- [ ] `ENABLE_SWAGGER=false` (don't expose API docs publicly)
- [ ] `JWT_SECRET` is a strong random string (min 64 chars), not a dev placeholder
- [ ] `CORS_ORIGINS` lists exact production frontend URLs only
- [ ] Database is on a managed service (not your dev laptop)
- [ ] R2 bucket is private
- [ ] Razorpay live keys are set (`rzp_live_*`)
- [ ] Razorpay webhook is configured with correct URL and secret
- [ ] `npx prisma migrate deploy` run before server start
- [ ] All SMTP credentials are valid (test by triggering a password reset)
- [ ] SSL/HTTPS is enabled on all domains
- [ ] Admin password is changed from default

---

## Health Check

```
GET /health    → { "status": "ok" }
```

Use this URL for Railway/uptime monitoring.
