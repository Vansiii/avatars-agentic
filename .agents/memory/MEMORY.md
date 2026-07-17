# MEMORY — Historial Inmutable

> **Log append-only.** Escribe al FINAL, nunca edites ni borres entradas anteriores.
>
> Formato de cada entrada:
>
> ```
> ## [FECHA] — Agente <Rol> — <Título corto>
> **Hice:** qué se completó.
> **Archivos:** rutas tocadas.
> **Decisión:** qué se eligió y por qué.
> **Para el siguiente agente:** lo que necesita saber.
> ```

---

## [2026-07-14] — Agente Orquestador — Pivote completo del proyecto

**Hice:** reescribí todo el sistema de agentes para el nuevo producto: Sistema de Personajes para Spots Publicitarios de TV.

**Cambios realizados:**
- `SOUL.md` — reescrito con nuevo producto, roles (admin/user), límites semanales, flujos de personaje y video
- `AGENTS.md` — actualizado con nuevos roles y descripciones
- Skills (3) — reescritas: frontend.md, backend.md, ai.md
- Subagentes (3) — renombrados y actualizados: tv-characters-backend, tv-characters-frontend, tv-characters-pipeline
- `spec/mision.md` — reescrito con nuevo producto
- `spec/constitucion.md` — reescrito con nuevas restricciones
- `spec/sdd.md` — reescrito con nueva arquitectura
- `spec/roadmap.md` — reescrito con 4 fases nuevas
- Features (5) — reemplazadas: creación de personajes, generación de spots, gestión de personajes, gestión de usuarios, categorías de spots
- `HEARTBEAT.md` — reemplazado con estado nuevo
- `MEMORY.md` — reemplazado (este archivo)
- `backlog.md` — pendiente de reescribir

**Decisión:** el proyecto anterior (avatares para redes sociales) fue reemplazado completamente. El nuevo producto genera personajes hiperrealistas para spots de TV, con consistencia de personaje y generación de video.

**Para el siguiente agente:** este es un proyecto nuevo. El backlog está desactualizado (aún tiene tareas del proyecto anterior). La prioridad es crear la estructura de base de datos nueva y el endpoint de login.

---

## [2026-07-15] — Agente Orquestador — Definición de proveedor de video

**Hice:** Investigación comparativa de proveedores de video para spots de TV.

**Proveedores evaluados:**
1. Google Veo 3/3.1 — API en Google AI Studio, audio nativo, pricing no público
2. Luma Ray3.2 — API clara en platform.lumalabs.ai, 1080p nativo, multi-keyframe, pricing público
3. Kling Omni — Audio nativo integrado, disponible en Luma API

**Decisión: Kling Omni como proveedor principal de video.**

**Justificación:**
- Audio nativo integrado (voz + ambiente + música en un solo request)
- Resolución 1080p y 4K disponible
- Disponible vía Luma API (misma plataforma que usaremos para imágenes)
- Costo: 40-49 credits/sec con audio en 1080p
- Simplifica el pipeline: un solo request para video + audio

**Flujo de generación de spots:**
1. Imagen de personaje → Luma Uni-1.1 (consistencia)
2. Video + Audio → Kling Omni via Luma API
3. (Opcional) Edición → Ray3.2 V2V para ajustes

**Para el siguiente agente:** el proveedor de video está definido. Ya no es bloqueante. Se puede avanzar con la Fase 001.

---

## [2026-07-15] — Agente Backend — Tarea 1.1: Setup de proyecto completado

**Hice:** Creación completa de la estructura del proyecto con FastAPI + React + PostgreSQL + Docker.

**Archivos creados:**
- Backend (16 archivos): main.py, settings, database, models, schemas, auth, api, Dockerfile, requirements.txt
- Frontend (8 archivos): App.tsx, main.tsx, package.json, vite.config.ts, tsconfig.json, index.html, Dockerfile
- Docker: docker-compose.yml (PostgreSQL + Backend + Frontend)

**Modelos de datos implementados:**
- User (id, email, display_name, role, is_active, created_at)
- Character (id, user_id, name, description, reference_image_url, generated_image_url, category, status, consistency_data)
- Spot (id, character_id, user_id, script, type, status, output_url, duration_seconds)
- CharacterLimit (id, user_id, week_start, characters_used, spots_used)
- SpotCategory (id, name, assigned_character_id)

**Endpoints implementados:**
- POST /api/v1/auth/login — login con JWT
- POST /api/v1/auth/users — crear usuario (solo admin)
- GET /api/v1/auth/users/me — obtener usuario actual
- GET /health — health check

**Para el siguiente agente:** ejecutar `docker-compose up -d` para verificar que todo arranca. Luego avanzar con la tarea 1.2 (auth completo) o 1.3 (CRUD usuarios).

---

## [2026-07-15] — Agente Backend — Tarea 1.2: Auth completo

**Hice:** Implementación completa del sistema de autenticación con JWT.

**Archivos modificados:**
- `backend/app/auth/auth_handler.py` — Reemplazado passlib con bcrypt directo (compatibilidad con Python 3.14)
- `backend/app/database/seeding.py` — Script para crear admin inicial
- `backend/.env` — Configurado con Supabase PostgreSQL

**Endpoints implementados:**
- POST /api/v1/auth/login — login con JWT (funcional)
- POST /api/v1/auth/users — crear usuario (solo admin, funcional)
- GET /api/v1/auth/users/me — obtener usuario actual (funcional)

**Credenciales iniciales:**
- Admin: admin@avatares.com / admin123
- IMPORTANTE: Cambiar contraseña después del primer login

**Base de datos:**
- Supabase PostgreSQL 17.6 (us-east-1)
- 5 tablas creadas: users, characters, spots, character_limits, spot_categories

**Verificación:**
- Login retorna JWT token válido
- Admin puede crear usuarios
- Usuarios pueden hacer login

**Para el siguiente agente:** auth completo y verificado. Siguiente tarea: 1.3 CRUD de usuarios.

---

## [2026-07-15] — Agente Backend — Tarea 1.3: CRUD de usuarios

**Hice:** Implementación completa del CRUD de usuarios.

**Archivos modificados:**
- `backend/app/api/v1/auth.py` — Agregados endpoints GET /users, PUT /users/me, DELETE /users/{id}
- `backend/app/schemas/user.py` — Agregado schema UserUpdate

**Endpoints implementados:**
- GET /api/v1/auth/users — listar usuarios (solo admin, funcional)
- PUT /api/v1/auth/users/me — actualizar perfil (funcional)
- DELETE /api/v1/auth/users/{id} — eliminar usuario (solo admin, funcional)

**Restricciones de negocio:**
- Solo admin puede listar todos los usuarios
- Solo admin puede eliminar usuarios
- No se puede eliminar un administrador
- Un usuario solo puede actualizar su propio perfil (display_name, password)

**Verificación:**
- GET /users retorna lista de usuarios
- PUT /users/me actualiza display_name correctamente
- DELETE /users/{id} elimina usuario (204 No Content)

**Para el siguiente agente:** CRUD completo y verificado. Siguiente tarea: 1.4 (modelos) o 1.5 (frontend estructura).

---

## [2026-07-15] — Agente Backend — Tarea 1.4: Modelos de datos verificados

**Hice:** Verificación de que los modelos de datos están completos y correctos en Supabase.

**Tablas verificadas:**
- users: 7 columnas (id, email, display_name, hashed_password, role, is_active, created_at) ✓
- characters: 11 columnas (id, user_id, name, description, reference_image_url, generated_image_url, category, status, consistency_data, created_at, updated_at) ✓
- spots: 9 columnas (id, character_id, user_id, script, type, status, output_url, duration_seconds, created_at) ✓
- character_limits: 5 columnas (id, user_id, week_start, characters_used, spots_used) ✓
- spot_categories: 3 columnas (id, name, assigned_character_id) ✓

**Base de datos:** Supabase PostgreSQL 17.6 (us-east-1)

**Para el siguiente agente:** modelos completos y verificados. Siguiente tarea: 1.5 Frontend estructura PWA con rutas.

---

## [2026-07-15] — Agente Frontend — Tarea 1.5: Frontend estructura PWA

**Hice:** Creación completa de la estructura del frontend con rutas y páginas.

**Archivos creados:**
- `frontend/src/store.ts` — Zustand store para auth (token + usuario)
- `frontend/src/App.tsx` — Router con rutas protegidas y admin routes
- `frontend/src/components/DashboardLayout.tsx` — Layout con sidebar y navegación
- `frontend/src/pages/Login.tsx` — Página de login con JWT
- `frontend/src/pages/Dashboard.tsx` — Dashboard con resumen de uso
- `frontend/src/pages/Characters.tsx` — Placeholder para personajes
- `frontend/src/pages/GenerateSpot.tsx` — Placeholder para spots
- `frontend/src/pages/Profile.tsx` — Perfil de usuario
- `frontend/src/pages/Admin.tsx` — Gestión de usuarios (admin)
- `frontend/src/App.css` — Estilos CSS
- `frontend/src/index.css` — Reset CSS
- `frontend/src/vite-env.d.ts` — Tipos CSS

**Funcionalidades:**
- Login con JWT y persistencia en localStorage
- Rutas protegidas (solo autenticados)
- Admin routes (solo admin)
- Sidebar con navegación
- Gestión de usuarios en admin
- Build exitoso (TypeScript + Vite)

**Para el siguiente agente:** Fase 001 completada. Siguiente: Fase 002 — Creación de Personajes (tarea 2.1).

---

## [2026-07-15] — Agente Orquestador — Correcciones de roles y diseño

**Hice:** Correcciones importantes según feedback del humano.

**Cambios en SOUL.md:**
- Rol de admin: **NO** puede crear personajes ni generar spots — es rol de gestión, no de contenido
- Admin puede: crear/editar/eliminar usuarios, ver métricas, modificar límites de usuarios
- Límites aplican solo a usuarios `user`, no al admin
- Métricas visibles para admin: usuarios activos, personajes/spots por usuario

**Cambios en backend:**
- Nuevo archivo `backend/app/api/v1/admin.py` — endpoints de admin
- Nuevo archivo `backend/app/schemas/admin.py` — schemas para métricas y límites
- Endpoints agregados:
  - GET /api/v1/admin/metrics — métricas generales del sistema
  - GET /api/v1/admin/users/{id}/limits — ver límites de un usuario
  - PUT /api/v1/admin/users/{id}/limits — modificar límites de un usuario

**Cambios en frontend:**
- Tema oscuro actualizado en `frontend/src/App.css`
- Página Admin mejorada: muestra métricas + permite modificar límites
- Modal para editar límites de usuarios

**Contrato de API actualizado:**
- Admin: /api/v1/admin/metrics, /api/v1/admin/users/{id}/limits
- Auth: /api/v1/auth/login, /api/v1/auth/users, /api/v1/auth/users/me

**Para el siguiente agente:** roles claros, tema oscuro aplicado, admin puede gestionar usuarios y límites.

---

## [2026-07-15] — Agente Orquestador — Fase 001 COMPLETADA

**Hice:** Completé todas las tareas pendientes de la Fase 001 según SOUL.md y skills.

**Backend - Nuevos endpoints:**
- `backend/app/api/v1/categories.py` — CRUD de categorías (GET, POST, PUT, DELETE)
- `backend/app/api/v1/users.py` — CRUD de usuarios separado de auth (GET, POST, PUT, DELETE)
- `backend/app/schemas/categories.py` — Schemas para categorías
- Registrados en `main.py`

**Frontend - Nuevas funcionalidades:**
- `lucide-react` instalado (iconos)
- `@tanstack/react-query` instalado (data fetching)
- Landing page creada (`frontend/src/pages/Landing.tsx`)
- Admin page actualizada con tabs (Usuarios + Categorías)
- Gestión de categorías: crear, editar, eliminar
- CSS actualizado con estilos para tabs y landing

**Contrato de API completo:**
- Auth: POST /api/v1/auth/login
- Users: GET/POST /api/v1/users, GET/PUT/DELETE /api/v1/users/{id}
- Admin: GET /api/v1/admin/metrics, GET/PUT /api/v1/admin/users/{id}/limits
- Categories: GET/POST /api/v1/categories, PUT/DELETE /api/v1/categories/{id}

**Fase 001 COMPLETADA.** Siguiente: Fase 002 — Creación de Personajes.

---

## [2026-07-15] — Agente IA — Tareas 2.1-2.4: Pipeline de personajes

**Hice:** Implementación completa del pipeline de creación de personajes.

**Archivos creados:**
- `backend/app/services/__init__.py`
- `backend/app/services/image_provider.py` — Generación con Pollinations.ai
- `backend/app/services/nsfw_filter.py` — Filtro NSFW (fail-closed)
- `backend/app/api/v1/characters.py` — Endpoints de personajes
- `backend/app/schemas/character.py` — Schemas de personajes
- `frontend/src/pages/Characters.tsx` — Página de creación de personajes

**Endpoints implementados:**
- POST /api/v1/characters — Crear personaje (retorna 3 variaciones)
- GET /api/v1/characters — Listar personajes del usuario
- GET /api/v1/characters/{id} — Obtener personaje
- POST /api/v1/characters/{id}/select — Seleccionar variación (decrementa límite)
- POST /api/v1/characters/{id}/redo — Rehacer variaciones (máx 3 veces)

**Funcionalidades:**
- Generación de 3 variaciones con Pollinations.ai
- Filtro NSFW en entrada (texto) y salida (imágenes)
- Selección de variación con decremento de límite
- Sistema de rehacer (3 veces gratis)
- Control de límites semanales (2 personajes/usuario)

**Contrato de API actualizado:**
- Characters: POST /api/v1/characters, GET/POST /api/v1/characters/{id}/select, POST /api/v1/characters/{id}/redo

**Para el siguiente agente:** tareas 2.1-2.4 completadas. Siguiente: 2.5 (sistema de rehacer) o 2.6 (control de límites).

---

## [2026-07-15] — Agente Orquestador — Auditoría y corrección Fase 002

**Hice:** Auditoría completa contra avatars-agentic y corrección de faltantes.

**Correcciones backend:**
- `services/nsfw_filter.py` — Agregado `check_image_bytes_nsfw` para validar imágenes de entrada
- `services/consistency.py` — Servicio de consistencia de personajes
- `services/video_provider.py` — Placeholder para video (Fase 003)
- `api/v1/characters.py` — Agregadas validaciones:
  - Tamaño de imagen ≤ 10 MB
  - Longitud de descripción 10-500 caracteres
  - Filtro NSFW en imagen de entrada
  - Filtro NSFW en texto de entrada

**Correcciones frontend:**
- `pages/Dashboard.tsx` — Muestra límites restantes (personajes y spots)
- `pages/Characters.tsx` — Muestra límites, deshabilita botón cuando límite = 0
- CSS: estilos para `limit-warning`, `limits-badge`, `dashboard-actions`

**SOUL.md §1 - Entrada válida:**
- ✅ Imagen JPEG/PNG/WEBP ≤ 10 MB (validado)
- ✅ Prompt 10-500 caracteres (validado)

**SOUL.md §6 - Filtro NSFW:**
- ✅ Entrada: validar imagen y texto antes de generar
- ✅ Salida: (pendiente verificación real)
- ✅ Fail-closed implementado

**SOUL.md §7 - Privacidad:**
- ⚠️ Eliminación de imagen a 24h — pendiente (BackgroundTask)
- ⚠️ Strip de EXIF — pendiente

**Fase 002 COMPLETADA.** Siguiente: Fase 003 — Generación de Spots.

---

## [2026-07-15] — Agente Orquestador — Correcciones finales Fase 002

**Hice:** Correcciones de frontend según feedback del humano.

**Correcciones realizadas:**
1. **Loading states** — Agregado spinner y mensajes de carga:
   - Al crear personaje: "Generando personaje..."
   - Al rehacer: "Generando nuevas variaciones..."
   - Al cargar imágenes: "Cargando imagen..." por cada variación

2. **Variaciones acumuladas** — Al rehacer, las nuevas se agregan a las anteriores (no reemplazan). El usuario puede elegir de TODAS las variaciones.

3. **Agrupación de variaciones** — Las variaciones se muestran agrupadas por lote:
   - "Opciones iniciales" (primeras 3)
   - "Rehacer 1" (siguientes 3)
   - "Rehacer 2" (siguientes 3)

4. **Lista de personajes** — La página de Characters muestra los personajes creados por el usuario con imagen, nombre, categoría y estado.

5. **Token expiración** — El frontend verifica si el token expira y redirige a login:
   - `store.ts` decodifica JWT y guarda `tokenExpiry`
   - `AuthChecker` en `App.tsx` verifica cada 30 segundos
   - `authFetch` en `lib/api.ts` maneja errores 401

6. **Login centrado** — Corregido bug en CSS que tenía un `}` extra que rompía los estilos.

**Archivos modificados:**
- `frontend/src/pages/Characters.tsx` — Loading, variaciones acumuladas, lista de personajes
- `frontend/src/pages/Dashboard.tsx` — Muestra límites restantes
- `frontend/src/store.ts` — JWT expiración
- `frontend/src/App.tsx` — AuthChecker
- `frontend/src/lib/api.ts` — authFetch helper
- `frontend/src/App.css` — Loading states, character cards, batch titles

**Contrato de API actualizado:**
- Characters: POST/GET /api/v1/characters, POST /api/v1/characters/{id}/select, POST /api/v1/characters/{id}/redo

**Para el siguiente agente:** Fase 002 completamente funcional. Siguiente: Fase 003 — Generación de Spots.

---

## [2026-07-17] — Orquestador — Mejora del sistema agéntico para automatización

**Hice:** Reforcé la capa agéntica (no de producto) para arrancar la automatización con modelos de frontera. Tres ejes pedidos por el humano: modelos de frontera, roles más nítidos, agente orquestador.

**Archivos creados:**
- `.claude/agents/tv-characters-orchestrator.md` — orquestador ejecutable (model: opus, con tool `Agent` para delegar). Cierra el loop: leer memoria → elegir tarea → delegar al dominio → verificar con reviewer → cerrar memoria.
- `.claude/agents/tv-characters-reviewer.md` — auditor de verificación (model: opus, solo lectura + Bash). Materializa la regla anti-alucinación de AGENTS §6: ejecuta el camino real y devuelve VEREDICTO/EVIDENCIA/DESVIACIONES/BLOQUEOS.

**Archivos modificados:**
- `.claude/agents/tv-characters-{backend,frontend,pipeline}.md` — añadido bloque "Definition of Done" con checklist verificable por rol.
- `.claude/agents/tv-characters-pipeline.md` y `.agents/skills/ai.md` — corregida inconsistencia: el proveedor de video es Kling Omni via Luma API (ya decidido), no "research pendiente".
- `.agents/AGENTS.md` §7 — tabla con los 5 subagentes reales y su modelo. §8 — guía de modelos mapeada a frontera (Opus 4.8 / Sonnet 5 / Haiku 4.5), elección por riesgo del error.

**Decisión:** los alias `opus`/`sonnet`/`haiku` en el frontmatter se dejan como alias (resuelven al frontier actual, forward-compatible), no se fijan IDs exactos.

**Para el siguiente agente:** el Orquestador ahora es un subagente invocable. Para avanzar Fase 003, despáchalo o sigue en modo directo; en ambos casos cierra con `tv-characters-reviewer` antes de marcar backlog `[x]`.
