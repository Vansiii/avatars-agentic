# Backlog — Sistema de Personajes para Spots de TV

> El Orquestador elige de aquí la siguiente tarea desbloqueada y marca `[ ]` → `[x]` al terminarla.
> **Regla:** una tarea solo se marca cuando se ha **verificado ejecutándola**.

**Leyenda:** 🔴 bloqueante · 🟡 importante · 🟢 mejorora

---

## Fase 001 — Base del Sistema 🔴

- [x] **1.1** 🔴 · Setup de proyecto: FastAPI + React + PostgreSQL con Docker
- [x] **1.2** 🔴 · Auth: login de admin con JWT (registro deshabilitado, admin crea usuarios)
- [x] **1.3** 🔴 · CRUD de usuarios: admin crea/elimina usuarios, usuario ve su perfil
- [x] **1.4** 🔴 · Modelos de datos: users, characters, spots, character_limits, spot_categories
- [x] **1.5** 🔴 · Frontend: estructura PWA con rutas (Login, Dashboard, Characters, Spots, Admin)

---

## Fase 002 — Creación de Personajes 🔴

- [x] **2.1** 🔴 · Endpoint POST /characters (imagen +/o texto + nombre + categoría)
- [x] **2.2** 🔴 · Pipeline de generación de imagen hiperrealista con Pollinations.ai
- [x] **2.3** 🔴 · Filtro NSFW en entrada y salida (fail-closed)
- [x] **2.4** 🔴 · Selección de variación (3 opciones)
- [x] **2.5** 🟡 · Sistema de rehacer (3 veces gratis, no decrementa límite)
- [x] **2.6** 🔴 · Control de límites semanales (2 personajes/usuario/semana)
- [x] **2.7** 🟡 · Frontend: página de crear personaje con flujo completo
- [x] **2.8** 🟡 · Consistencia: guardar reference_image_url para uso en videos

### Correcciones Fase 002 (2026-07-15):
- [x] Loading states (crear, rehacer, cargar imágenes)
- [x] Variaciones acumuladas (anteriores + nuevas)
- [x] Agrupación de variaciones por lote
- [x] Lista de personajes creados
- [x] Token expiración → redirige a login
- [x] Login centrado (CSS corregido)

---

## Fase 003 — Generación de Spots (Video) 🔴

- [ ] **3.1** 🔴 · Endpoint POST /spots (personaje + script + tipo)
- [ ] **3.2** 🔴 · Pipeline de generación de video (Kling Omni via Luma API)
- [ ] **3.3** 🔴 · Consistencia: personaje aparece en el video (usar reference_image_url)
- [ ] **3.4** 🟡 · Tipos de video: short (3-5s) y long (15-30s)
- [ ] **3.5** 🟡 · Selección de variación de video (3 opciones)
- [ ] **3.6** 🟡 · Sistema de rehacer para spots (máx 3 veces)
- [ ] **3.7** 🔴 · Control de límites semanales (5 spots/usuario/semana)
- [ ] **3.8** 🟡 · Frontend: página de generar spot con flujo completo

---

## Fase 004 — Cierre del Alpha 🟢

- [ ] **4.1** 🟡 · Gestión de personajes: editar, eliminar, reasignar categoría
- [ ] **4.2** 🟡 · Categorías de spots predefinidas (deportes, noticias, etc.)
- [ ] **4.3** 🟢 · Dashboard con métricas de uso (admin)
- [ ] **4.4** 🟢 · Tests E2E del flujo completo
- [ ] **4.5** 🟢 · README de arranque: cómo levantar el sistema

---

## Deuda técnica conocida

- Consistencia de personaje por imagen de referencia (no modelos avanzados) — suficiente para Alpha
- ~~Proveedor de video por definir (research pendiente)~~ ✅ Kling Omni via Luma API (2026-07-15)
- `create_all` en arranque en vez de Alembic — aceptable para Alpha
- NSFW filter: implementación básica con palabras clave (Alpha) — en producción usar NudeNet o similar
- Eliminación de imágenes a 24h y strip de EXIF — pendiente para Fase 004

---

## Notas para el siguiente agente

### Cómo levantar el sistema
```bash
# Backend
cd backend
.\.venv\Scripts\activate
uvicorn app.main:app --reload --port 8000

# Frontend
cd frontend
npm run dev
```

### Credenciales
- Admin: admin@avatares.com / admin123

### Stack
- Backend: Python 3.11 + FastAPI + SQLAlchemy + PostgreSQL (Supabase)
- Frontend: React 19 + TypeScript + Vite + zustand + react-query

### Proveedor de video para Fase 003
- **Kling Omni via Luma API**
- Audio nativo integrado
- 1080p / 4K
- Documentado en MEMORY.md (entrada del 2026-07-15)
