# 4R — Revisión Backend y Frontend

**Fecha:** 2026-07-20
**Scope:** avatars-backend (FastAPI/Python) + avatars-frontend (React/TypeScript)
**Estado:** Post force-push reset, repos fresco desde origin/main.

---

## Backend — CRITICAL

| # | Hallazgo | Lens | Archivo |
|---|---|---|---|
| 1 | **SECRET_KEY default hardcodeado** `"cambiar-esta-clave-en-produccion"` — si falta .env, cualquiera forja JWTs | Risk | `config/settings.py:6` |
| 2 | **NSFW filter de imágenes es no-op** — solo checkea tamaño >1KB, toda imagen real pasa sin análisis | Risk | `services/nsfw_filter.py:57-69` |
| 3 | **DDL y seed ejecutados a nivel de módulo** (`create_all` + `seed_default_categories` en import time) — race conditions con múltiples workers | Risk | `main.py:15-16` |
| 4 | **`role` es `str` sin validación** — cualquiera puede crear usuarios con role "admin", no hay enum ni constraint en DB | Risk | `schemas/user.py:10`, `models/models.py:21` |
| 5 | **Sin rate limiting en ningún endpoint** — fuerza bruta ilimitada contra login y cualquier endpoint | Risk | `api/v1/auth.py:24` |

## Backend — WARNING

- Refresh tokens no rotativos ni revocables (ventana de 7 días si roban uno)
- CORS con `allow_methods=['*']` y `allow_headers=['*']` innecesariamente permisivo
- Sin middleware de manejo de errores global — excepciones no manejadas exponen detalles internos
- Contraseña admin hardcodeada `admin123` sin forzar cambio en primer login
- Dockerfile en prod con `--reload` (antipatrón)
- Endpoint PUT `/users/me` cambia contraseña sin pedir la actual
- Sin `pool_pre_ping` en el pool de SQLAlchemy — conexiones muertas tras failover
- Conexiones DB retenidas durante polls de HeyGen de hasta 15 minutos (agota el pool)
- `generate_spot_variations` envía submits secuenciales, no paralelos (docstring miente)
- No retry/backoff ante fallos de HeyGen — un timeout aborta todo el spot
- `check_text_nsfw` usa substring matching → "Sussex" bloqueado por "sex", "exxon" por "xxx"
- `consistency.py` completo es dead code (nunca importado)
- `video_provider.py` con estado global mutable (`_voice_id_cache` + `asyncio.Lock`)
- `security.log` FileHandler duplicado en cada recarga por `--reload`
- LoginRequest.email usa `str` en vez de `EmailStr` (pydantic ya importado)
- `import base64` dentro del cuerpo de una función en vez de top-level
- Modelos SQLAlchemy sin `__repr__` (debug inútil)
- `redos_remaining=3` hardcodeado como magic number
- No existen tests (carpeta `tests/` vacía/ausente)

---

## Frontend — CRITICAL

| # | Hallazgo | Lens | Archivo |
|---|---|---|---|
| 1 | **JWTs en localStorage** via Zustand persist — cualquier XSS roba tokens. Access + refresh tokens + user PII en `localStorage` clave `avatares-auth` | Risk | `src/store.ts:40-81` |

## Frontend — WARNING

- **Dockerfile corre `npm run dev` en producción** — Vite dev server con HMR, source maps, sin build
- **Sin Content-Security-Policy** — sin defensa en profundidad contra XSS
- Errores del backend renderizados directamente al DOM (sin sanitizar)
- **God-components**: `Characters.tsx` (681 líneas, 17 useState) y `Admin.tsx` (511 líneas, 13 useState)
- **Tipos duplicados**: `Character`, `Category`, `User` definidos localmente en 2-3 archivos cada uno con formas distintas
- **TanStack Query configurado pero no usado** — 4 páginas implementan el mismo patrón manual `useEffect + useState + try/catch`
- Arrow functions referenciadas antes de su declaración
- **Errores de API silenciados** — 4xx/5xx se muestran como "no tenés nada todavía"
- **Sin ruta 404** — URLs inválidas renderizan pantalla vacía
- Botón de crear usuario permite envíos duplicados (sin estado de submitting)
- Object URLs nunca liberadas (`URL.createObjectURL`) — fuga de memoria
- `authFetch` sin timeout/AbortController — requests colgadas indefinidamente
- Media (`<img>`, `<video>`) sin `onError` fallback → UI rota si falla URL
- Dashboard muestra cuotas sintéticas cuando la API falla
- Login sin intent counter ni feedback de rate limiting
- Passwords en React state visibles en DevTools

---

## Ambos repos

- **0 tests.** Backend sin tests, frontend sin tests. Cualquier cambio se deploya sin verificación.
- El NSFW filter de texto tiene falsos positivos masivos por substring matching.
- Sin health check real (solo `{"status": "ok"}` sin verificar DB ni dependencias).

---

## Hallazgos completos

Ver archivos de revisión individuales en cada repo:

- Backend: `review-risk`, `review-readability`, `review-reliability`, `review-resilience`
- Frontend: `review-risk`, `review-readability`, `review-reliability`, `review-resilience`
