# ADR: Session Controls for Avatars Backend (PR2)

- **Date:** 2026-07-16
- **Status:** Implemented (commit `80c01f0` on `feature/security-p1/pr2-session`)
- **Scope:** Session authority model for the avatars-backend project

## Context

PR2 of avatars-backend adds request throttling and JWT session revocation enforceable across backend instances. This PR closes the gap left after PR1: Redis is available, but not yet used to coordinate rate limits or invalidate access tokens.

## Decision

**PostgreSQL is the durable authority for session state. Redis is a fast, disposable projection.**

### Architecture

```
PostgreSQL (authority)
  ├── sessions (sid, user_id, revoked, expires_at)
  ├── refresh_tokens (hash, user_id, session_id, replaced_by)
  └── users (is_admin)
        │
        ▼ degraded mode (Redis down)
Redis (fast projection)
  ├── blacklist:v1:sid:{sid} → revoked cache
  ├── ratelimit:v1:* → rate-limit counters
  └── rebuilt from PG on recovery
```

### Degraded mode

When Redis is unavailable:

- Access tokens issued with 2-minute TTL (instead of 15–30 min)
- Blacklist verified against `sessions.revoked` in PostgreSQL
- Rate limiter falls back to per-process `LocalWindowLimiter` (conservative caps)
- Background recovery probe every 30 seconds
- On Redis recovery: cache repopulated from PG, normal mode restored

### Rate limiting

- Atomic Lua `INCR` + `EXPIRE` via Redis `SCRIPT LOAD`/`EVALSHA`
- Dual limits: general (100 req/min) and generation (20 gen/min)
- Identity: JWT `sub` (authenticated) or client IP (anonymous)
- `TRUST_PROXY_HEADERS` default-off for spoof protection

### Session lifecycle

- **Login**: creates `Session` + `RefreshToken` in PG transaction
- **Logout**: revokes session in PG + best-effort Redis cache
- **Refresh**: atomic conditional UPDATE + INSERT in PG transaction
- **Legacy upgrade**: pre-PR2 tokens get one-time upgrade with hash tombstone

## Key files

- `app/middleware/rate_limit.py` — Redis Lua rate limiter + LocalWindowLimiter
- `app/middleware/degraded.py` — degraded state machine + recovery
- `app/auth/auth_handler.py` — JWT with sid, jti, iat claims
- `app/auth/dependencies.py` — token claims + blacklist verification
- `app/api/v1/auth.py` — login, refresh rotation, logout, legacy upgrade
- `app/models/session.py` — Session SQLAlchemy model
- `app/models/refresh_token.py` — RefreshToken SQLAlchemy model
- `alembic/versions/002_pr2_session_controls.py` — migration
