# Secure MERN Auth + RBAC

A small MERN baseline with a production-oriented authentication/authorization layer.

## Security properties

- Server-side sessions stored in MongoDB; the browser only gets an opaque session cookie.
- `HttpOnly`, `SameSite=Lax`, and `Secure` cookies in production.
- Session ID regeneration on login to reduce session fixation risk.
- Password hashing with bcrypt.
- Login rate limiting plus a second generic API limiter.
- Helmet security headers.
- Strict JSON body-size limit.
- Zod validation for auth and resource inputs.
- Generic login errors to avoid username/email enumeration.
- Authorization is deny-by-default and centralized in `server/src/middleware/authz.js`.
- Resource reads/writes enforce ownership server-side; clients cannot select another owner.
- Role changes are never accepted from client input.
- Admin-only route demonstrates RBAC.
- Logout destroys the session server-side.
- No secrets committed; `.env.example` is provided.
- No Mongo `_id` is trusted from the client for ownership.
- CORS is restricted to the configured client origin and credentials are required.

## Requirements

- Node.js 20+
- MongoDB 7+ (or MongoDB Atlas)

## Run

### 1. Start MongoDB

```bash
docker compose up -d mongo
```

### 2. Server

```bash
cd server
cp .env.example .env
npm install
npm run dev
```

### 3. Client

```bash
cd client
npm install
npm run dev
```

Open the Vite URL shown in the terminal.

## Demo flow

1. Register two accounts.
2. Log in as account A and create a note.
3. Log out and log in as account B.
4. The API will not return account A's note, even if account B submits its ID.
5. `/api/admin/users` returns `403` for a normal user.
6. Sending `"role":"admin"` during registration or profile updates has no effect because role assignment is server-controlled.
7. Repeated failed login attempts are rate-limited.

## Production checklist

Before deploying:

- Set a long random `SESSION_SECRET`.
- Use HTTPS and keep `COOKIE_SECURE=true`.
- Set `CLIENT_ORIGIN` to the exact deployed frontend origin.
- Use a managed MongoDB deployment with authentication, TLS, backups, and network restrictions.
- Put the API behind a reverse proxy/load balancer and configure `TRUST_PROXY` appropriately.
- Replace the demo in-memory operational assumptions with centralized observability/alerting.
- Add email verification, password reset, MFA, audit logging, and session/device management if required by the product.
- Configure a persistent, shared rate-limit store if running multiple API instances.
- Do not put bearer tokens or session IDs in localStorage.
