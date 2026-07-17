# HEARTBEAT — El Pulso del Proyecto

> **Foto del estado ACTUAL.** Este archivo se **sobrescribe** en cada cambio de turno.
> No es un historial: si buscas qué pasó antes, ve a `MEMORY.md`.

**Última actualización:** 2026-07-17 — por el **Agente Orquestador** (auditoría + corrección de bugs reales en Fase 002 + rediseño profesional del frontend para Canal 11 TVU · UAGRM)

---

## Estado Global del Proyecto

**Nuevo producto:** Sistema de Personajes para Spots Publicitarios de TV

| Fase | Estado |
|------|--------|
| Fase 001 — Base del sistema | ✅ COMPLETA |
| Fase 002 — Creación de personajes | ✅ COMPLETA |
| Fase 003 — Generación de spots | ⏳ Pendiente |
| Fase 004 — Cierre del Alpha | ⏳ Pendiente |

---

## Bloqueos actuales

1. ~~**Proveedor de video:**~~ ✅ RESUELTO: Kling Omni via Luma API
2. ~~**Modelo de datos nuevo:**~~ ✅ RESUELTO: Modelos creados y verificados en Supabase
3. **Pollinations.ai (free tier) rate-limita bajo ráfagas de pedidos (429).** Descubierto al verificar el fix del filtro NSFW de salida (ver abajo): crear un personaje ahora puede tardar 30-90s+ porque el backend espera a que las 3 imágenes generen y se descarguen antes de responder (antes no esperaba nada, solo construía URLs). No es un bug de este cambio — es el costo real de cumplir SOUL.md §6 (validar salida) con un proveedor gratuito sin API key. Si esto molesta en uso real, la solución es conseguir una API key de Pollinations o mover el chequeo a `BackgroundTasks` con polling, no revertir el filtro.

---

## Estado Backend — Fase 002 auditada y corregida (2026-07-17)

Se encontraron y corrigieron bugs reales (no solo estilo) verificados ejecutando contra el Supabase real del proyecto:

- **NSFW de salida ahora se ejecuta de verdad.** `check_image_nsfw()` existía desde Fase 002 pero nadie la llamaba (confirmado por grep) — las variaciones se mostraban sin pasar el filtro de salida que SOUL.md §6 exige. Ahora `nsfw_filter.check_generated_images_nsfw()` descarga cada variación generada y la valida antes de devolverla; fail-closed real, con reintento ante 429.
- **Límites por usuario ahora son configurables de verdad.** Antes `PUT /admin/users/{id}/limits` manipulaba `characters_used` asumiendo que el tope siempre era 2 — subir el límite a 5 no cambiaba nada porque `characters.py` seguía comparando contra `2` hardcodeado. Se agregaron columnas `characters_limit_override`/`spots_limit_override` en `users` (migradas a mano en Supabase, `ALTER TABLE ... ADD COLUMN IF NOT EXISTS`, ver MEMORY.md) y un módulo único `app/services/limits.py`.
- **CRUD de usuarios duplicado eliminado.** `auth.py` solo tiene `login`/`me` ahora; `users.py` es el único CRUD admin. Verificado: `/api/v1/auth/users` → 404, `/api/v1/users` funciona.
- **Categorías desincronizadas, corregido.** `create_character` valida contra `SpotCategory` real (DB), no una lista hardcodeada — una categoría creada por el admin ya se puede usar. Se agregó `seed_default_categories()` (Deportes/Noticias/Entretenimiento) al arranque, idempotente.
- **CORS ahora sale de `settings.CORS_ORIGINS`**, no hardcodeado a localhost.

### Auth
- POST /api/v1/auth/login — login con JWT
- GET /api/v1/auth/users/me — obtener usuario actual
- PUT /api/v1/auth/users/me — actualizar perfil

### Users (Admin) — único CRUD, ya no duplicado con auth.py
- GET /api/v1/users — listar usuarios
- POST /api/v1/users — crear usuario
- GET /api/v1/users/{id} — obtener usuario
- PUT /api/v1/users/{id} — actualizar usuario
- DELETE /api/v1/users/{id} — eliminar usuario

### Admin
- GET /api/v1/admin/metrics — métricas del sistema
- GET /api/v1/admin/users/{id}/limits — ver límites efectivos (override o default)
- PUT /api/v1/admin/users/{id}/limits — modificar el override real (persiste en `users`)

### Categories
- GET /api/v1/categories — listar categorías (sembradas por defecto al arrancar)
- POST /api/v1/categories — crear categoría
- PUT /api/v1/categories/{id} — actualizar categoría
- DELETE /api/v1/categories/{id} — eliminar categoría

### Characters
- POST /api/v1/characters — crear personaje (retorna variaciones que ya pasaron NSFW de salida; puede tardar 30-90s+, ver Bloqueos)
- GET /api/v1/characters — listar personajes del usuario
- GET /api/v1/characters/{id} — obtener personaje
- POST /api/v1/characters/{id}/select — seleccionar variación
- POST /api/v1/characters/{id}/redo — rehacer variaciones (máx 3, también con NSFW de salida)

### Servicios
- `image_provider.py` — Generación con Pollinations.ai
- `nsfw_filter.py` — Filtro NSFW fail-closed: entrada Y salida (antes solo entrada estaba conectada)
- `services/limits.py` — **nuevo**: única fuente de verdad para límites semanales (antes duplicado en characters.py/admin.py)
- `consistency.py` — Consistencia de personajes
- `video_provider.py` — Placeholder para Fase 003

### Validaciones implementadas
- ✅ Tamaño imagen ≤ 10 MB
- ✅ Longitud descripción 10-500 caracteres
- ✅ Filtro NSFW en entrada Y salida (imagen + texto)
- ✅ Control de límites semanales, configurables por usuario (override real)

### DB
- Supabase PostgreSQL 17.6 (us-east-1)
- Credenciales en `backend/.env`
- **Nota Alpha:** `create_all()` no altera tablas existentes — al agregar columnas a modelos ya migrados hay que correr el `ALTER TABLE` a mano contra Supabase (se hizo para `characters_limit_override`/`spots_limit_override`). Sigue pendiente Alembic (deuda ya conocida).

---

## Estado Frontend — Rediseño profesional aplicado (2026-07-17)

### Stack
- React 19 + TypeScript + Vite
- zustand (estado global con persist)
- @tanstack/react-query
- lucide-react (iconos)
- Tema oscuro con identidad propia (sin logo real — placeholder textual, ver abajo)
- **`oxlint` instalado y con script `npm run lint` de verdad** — antes `.agents/skills/frontend.md` decía "ya configurado" pero no existía ni la dependencia ni el script.

### Identidad visual
- **Logos oficiales ya integrados** (el usuario los proveyó): `frontend/public/logo-tvu.jpg` (escudo Canal 11 TVU) y `logo-uagrm.jpg` (UAGRM) — estaban puestos en `frontend/src/public/` por error (esa carpeta NO es la estática de Vite, solo `frontend/public/` lo es); se movieron. Se muestran en Sidebar, Login y Landing dentro de un chip blanco (`.brand-logo-chip`) porque son JPEG con fondo blanco y se verían con un recuadro gris sobre el tema oscuro sin eso.
- Paleta actualizada a los **colores institucionales reales** (azul `#1a56db` + rojo `#d92626`, muestreados de los logos) — reemplaza el ámbar genérico usado antes de tener los assets.
- `index.html` `<title>` = "TVU Studio · Canal 11 UAGRM", favicon = `logo-tvu.jpg`.
- Marca textual "TVU Studio" se mantiene como acompañante de los logos, no reemplazo.

### Páginas
- **Landing** — nav superior + hero con marca + footer institucional
- **Login** — con marca "TVU Studio · Canal 11 UAGRM"
- **Dashboard** — cards de límites ahora con barra de progreso (usado/total), no solo texto
- **Characters** — mismo flujo, sin estilos inline, loading unificado (`Loader2` + texto)
- **GenerateSpot** — placeholder "Próximamente" rediseñado (antes era un botón deshabilitado suelto) — Fase 003 sigue sin implementarse, fuera de alcance de esta tarea
- **Profile** — sin cambios de lógica
- **Admin** — paleta nueva, loading unificado

### Correcciones de consistencia
- **`Dashboard.tsx` y `Admin.tsx` usaban `fetch` crudo** con header manual en vez de `authFetch` de `lib/api.ts` — la expiración de sesión (401 → logout + redirect) solo se aplicaba en `Characters.tsx`. Ya migradas ambas a `authFetch`.
- Mojibake corregido en `store.ts` (comentario con caracteres corruptos).

### Funcionalidades implementadas
- ✅ Login con JWT y persistencia
- ✅ Rutas protegidas (solo autenticados)
- ✅ Admin routes (solo admin)
- ✅ Token expiración → redirige a login (ahora consistente en TODAS las páginas, no solo Characters)
- ✅ Loading states unificados (crear, rehacer, cargar imágenes, listas)
- ✅ Variaciones acumuladas (anteriores + nuevas)
- ✅ Límites visibles con barra de progreso
- ✅ Lista de personajes creados
- ✅ Build exitoso (`tsc -b && vite build`) y lint limpio (`npx oxlint src`, 0 warnings)
- ✅ Verificado con dev server real: `npm run dev` sirve la app, el proxy `/api` a `:8000` funciona end-to-end
- ⚠️ No se tomaron capturas de navegador reales (sin herramienta de automatización de navegador en este entorno) — verificado a nivel HTTP/build/lint, no visual pixel-a-pixel. Recomendado: abrir manualmente y confirmar antes de dar por cerrado el rediseño.

---

## Estado IA — Completo para Fase 002

- Imágenes: Pollinations.ai (funcional)
- Video: Kling Omni via Luma API (pendiente Fase 003)
- NSFW: Filtro implementado (fail-closed)
- Consistencia: imagen de referencia guardada

---

## Credenciales

- **Admin:** admin@avatares.com / admin123
- **Backend:** http://localhost:8000 (Swagger: /docs)
- **Frontend:** http://localhost:5173

---

## Pendiente para Fase 003

- [ ] Endpoint POST /spots
- [ ] Pipeline de generación de video (Kling Omni via Luma API)
- [ ] Consistencia de personaje en video (usar reference_image_url)
- [ ] Tipos de video: short (3-5s) y long (15-30s)
- [ ] Selección de variación de video (3 opciones)
- [ ] Sistema de rehacer para spots (máx 3 veces)
- [ ] Control de límites semanales (5 spots/usuario/semana)
- [ ] Frontend: página de generar spot con flujo completo

---

## Para el siguiente agente

1. Leer SOUL.md antes de empezar
2. Para levantar: `backend/.venv` ya existe (venv creado y con `requirements.txt` instalado durante esta sesión) — `uvicorn app.main:app --reload` desde `backend/`
3. Frontend: `npm run dev` desde `frontend/` (ya tiene `node_modules` + `oxlint`)
4. Usar `authFetch` del archivo `frontend/src/lib/api.ts` para TODAS las llamadas autenticadas — ya no hay excepciones (Dashboard/Admin corregidos)
5. **Antes de tocar `characters.py` o `admin.py`**: los límites y el `week_start` ahora viven en `app/services/limits.py` — no los reintroduzcas duplicados
6. Siguiente fase: **Fase 003 — Generación de Spots (Video)** — sigue sin implementar, fuera de alcance de esta sesión (decisión explícita del usuario)
7. Proveedor de video: **Kling Omni via Luma API** (documentado en MEMORY.md)
8. Si agregas columnas a modelos existentes: `create_all()` no las migra en tablas ya creadas en Supabase — hay que correr el `ALTER TABLE` a mano (ver MEMORY.md de esta sesión para el patrón usado)
