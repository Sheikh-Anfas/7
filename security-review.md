# Security review checklist

## (a) Read another user's data

The note collection endpoint is scoped by the authenticated session:

`Note.find({ ownerId: req.user._id })`

Individual note access loads the resource and then checks ownership through the centralized authorization module. A normal `user` does not have `notes:read:any`.

The API never accepts `ownerId` from the client when creating a note.

## (b) Elevate `user` to `admin`

Registration uses:

`role: "user"`

and does not read a role from the request body. The same principle applies to all client-facing auth flows: the role in the session comes from the database record, not from a client-supplied field.

The authorization module recognizes only the explicit `user` and `admin` roles and denies unknown roles/actions.

## (c) Brute-force a password

`POST /api/auth/login` is behind a dedicated rate limiter: 10 failed attempts per 15-minute window per limiter key. Successful requests do not consume the failed-attempt budget.

The login response is generic for both a missing account and a wrong password.

Passwords are hashed with bcrypt (12 rounds).

## Additional OWASP-oriented controls

- Session IDs are regenerated after successful authentication.
- Session cookies are HttpOnly.
- SameSite is `Lax`.
- Secure cookies are enabled in production.
- Sessions are persisted in MongoDB, not MemoryStore.
- Helmet is enabled.
- JSON bodies are capped at 16 KB.
- Zod rejects unexpected fields with `.strict()`.
- CORS is restricted to the configured origin.
- Authentication and authorization are separate middleware concerns.
- Secrets are environment variables.
- Errors returned to clients do not expose stack traces.
- Resource IDs are validated as Mongo ObjectIds.
- Database queries are constructed server-side; no client-provided Mongo query operators are accepted.
- Role/permission logic is centralized in `server/src/middleware/authz.js`.

## Important production caveat

For multiple API instances, use a shared rate-limit store (for example Redis) so an attacker cannot bypass limits by distributing requests across instances. Also configure your reverse proxy's trusted IP headers correctly before relying on IP-based limiting.
