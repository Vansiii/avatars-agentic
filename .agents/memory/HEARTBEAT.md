# HEARTBEAT — El Pulso del Proyecto

> **Foto del estado ACTUAL.** Este archivo se **sobrescribe** en cada cambio de turno.
> No es un historial: si buscas qué pasó antes, ve a `MEMORY.md`.

**Última actualización:** 2026-07-17 — por el **Agente Orquestador** (mejora de la capa agéntica: orquestador + reviewer ejecutables, modelos de frontera, roles con Definition of Done)

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

---

## Estado Backend — Completo

### Auth
- POST /api/v1/auth/login — login con JWT
- GET /api/v1/auth/users/me — obtener usuario actual
- PUT /api/v1/auth/users/me — actualizar perfil

### Users (Admin)
- GET /api/v1/users — listar usuarios
- POST /api/v1/users — crear usuario
- GET /api/v1/users/{id} — obtener usuario
- PUT /api/v1/users/{id} — actualizar usuario
- DELETE /api/v1/users/{id} — eliminar usuario

### Admin
- GET /api/v1/admin/metrics — métricas del sistema
- GET /api/v1/admin/users/{id}/limits — ver límites
- PUT /api/v1/admin/users/{id}/limits — modificar límites

### Categories
- GET /api/v1/categories — listar categorías
- POST /api/v1/categories — crear categoría
- PUT /api/v1/categories/{id} — actualizar categoría
- DELETE /api/v1/categories/{id} — eliminar categoría

### Characters
- POST /api/v1/characters — crear personaje (retorna 3 variaciones)
- GET /api/v1/characters — listar personajes del usuario
- GET /api/v1/characters/{id} — obtener personaje
- POST /api/v1/characters/{id}/select — seleccionar variación
- POST /api/v1/characters/{id}/redo — rehacer variaciones (máx 3)

### Servicios
- `image_provider.py` — Generación con Pollinations.ai
- `nsfw_filter.py` — Filtro NSFW (fail-closed, valida imagen y texto)
- `consistency.py` — Consistencia de personajes
- `video_provider.py` — Placeholder para Fase 003

### Validaciones implementadas
- ✅ Tamaño imagen ≤ 10 MB
- ✅ Longitud descripción 10-500 caracteres
- ✅ Filtro NSFW en entrada (imagen + texto)
- ✅ Control de límites semanales

### DB
- Supabase PostgreSQL 17.6 (us-east-1)
- Credenciales en `backend/.env`

---

## Estado Frontend — Completo

### Stack
- React 19 + TypeScript + Vite
- zustand (estado global con persist)
- @tanstack/react-query
- lucide-react (iconos)
- Tema oscuro

### Páginas
- **Landing** — página de inicio
- **Login** — login con JWT, centrado
- **Dashboard** — muestra límites restantes
- **Characters** — lista de personajes + crear nuevo + selección de variaciones
- **GenerateSpot** — placeholder para spots
- **Profile** — perfil de usuario
- **Admin** — métricas + gestión usuarios + gestión categorías

### Funcionalidades implementadas
- ✅ Login con JWT y persistencia
- ✅ Rutas protegidas (solo autenticados)
- ✅ Admin routes (solo admin)
- ✅ Token expiración → redirige a login
- ✅ Loading states (crear, rehacer, cargar imágenes)
- ✅ Variaciones acumuladas (anteriores + nuevas)
- ✅ Límites visibles y botón deshabilitado
- ✅ Lista de personajes creados
- ✅ Build exitoso

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
2. El backend está corriendo en http://localhost:8000
3. El frontend está corriendo en http://localhost:5173
4. Usar `authFetch` del archivo `frontend/src/lib/api.ts` para llamadas autenticadas
5. Siguiente fase: **Fase 003 — Generación de Spots (Video)**
6. Proveedor de video: **Kling Omni via Luma API** (documentado en MEMORY.md)
