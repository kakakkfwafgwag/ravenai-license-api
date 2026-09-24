# RavenAI license / whitelist dashboard

Dark Next.js + Prisma/SQLite account and licensing starter with a desktop-client HWID key flow.

## Included

- Dashboard showing whitelist state, GOOD/BAD subscription status, HWID device, and time remaining
- **White List** license page
- Sole admin: `lauripalonen16@gmail.com`
- Admin page requires **both** `ADMIN` role and that exact email
- Generate 1–100 keys per request
- Keys stored in Prisma/SQLite and visible to the sole admin
- Product, owner, expiry, status, HWID and activation limit per key
- First desktop activation binds the supplied HWID
- Same HWID can activate/verify again; other HWIDs are rejected after the activation limit
- Configurable activation limit
- Admin Reset HWID and Revoke actions
- Expiration/revocation checks

## Setup

```bash
npm install
cp .env.example .env
npx prisma db push
npm run db:seed
npm run dev
```

Set a strong `AUTH_SECRET` and `ADMIN_PASSWORD` in `.env`. The seed always creates/updates `lauripalonen16@gmail.com` as the sole admin, marks it whitelisted, and gives it an initial active RavenAI Core license if none exists.

## Desktop client API

### Activate / bind HWID
`POST /api/license/activate`

```json
{"licenseKey":"RAVEN-...","hwid":"YOUR-STABLE-HWID","deviceName":"Gaming PC"}
```

The first activation stores the HWID. If the same key is presented with the same HWID it succeeds and refreshes `lastSeenAt`. A new HWID is accepted only while the configured activation limit has not been reached.

### Verify
`POST /api/license/verify`

```json
{"licenseKey":"RAVEN-...","hwid":"YOUR-STABLE-HWID"}
```

Successful response includes `valid`, product slug, status and expiry. Invalid/revoked/expired keys or an HWID mismatch return `valid: false` with a reason.

## Admin API

`POST /api/admin/licenses` body:

```json
{"email":"customer@example.com","productId":"...","count":10,"activationLimit":1,"durationDays":30}
```

`count` supports 1–100. Admin routes require an authenticated account whose role is `ADMIN` **and** whose normalized email exactly matches `lauripalonen16@gmail.com`.

Admin actions:
- `POST /api/admin/licenses/:id/reset-hwid`
- `POST /api/admin/licenses/:id/revoke`

## Production notes

Use HTTPS, a strong random `AUTH_SECRET`, rate limiting on public activation/verification endpoints, and PostgreSQL for multi-instance production. Keep SellAuth secrets server-side. Do not trust a client-provided HWID as a security boundary by itself; combine licensing with server-side checks and abuse controls.
