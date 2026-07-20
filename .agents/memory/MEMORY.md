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

## [2026-07-20] — Agente Orquestador — Research: adaptar HeyGen a formato noticiero

**Hice:** el usuario pidió investigar cómo mejorar la integración de HeyGen para que los spots de categoría "noticias" se vean como un noticiero real (no solo un talking-head plano). Analicé `video_provider.py`/`spots.py` actuales y confirmé contra la documentación oficial de HeyGen (developers.heygen.com, docs.heygen.com) qué falta.

**Hallazgo clave:** hoy se usa el endpoint plano `POST /v3/videos` (type=avatar/image) — solo avatar + guión + voz, sin fondo de estudio, sin rótulo inferior, sin logo, sin cintillo, sin escenas. HeyGen tiene un endpoint mejor para esto: **Template API** (`POST /v2/template/{template_id}/generate`) — se arma UNA plantilla a mano en el editor web de HeyGen (con set de noticiero completo) y por cada spot solo se mandan variables (`character`=avatar_id, `text`=guión, rótulo con nombre/cargo, `voice`, opcionalmente `image`/`video` de B-roll). HeyGen incluso vende esto como producto separado ("AI News Generator") con plantillas de sala de prensa, rótulos, cintillos. También soporta `caption`/`subtitles` para subtítulos automáticos.

**Decisión:** no implementar todavía — se anotó como tarea **4.6** en `backlog.md`, bloqueada hasta que el usuario arme la(s) plantilla(s) en HeyGen y provea el `template_id` (no se puede probar sin costo real). Se descartó explícitamente el Streaming/Interactive Avatar de HeyGen (avatar en vivo vía WebRTC) — es para chat en tiempo real, no para spots pregrabados, y su infra de sesión está fuera del alcance del Alpha (SOUL.md §8, monolito sin infra de tiempo real).

**Archivos:** `.agents/steering/backlog.md` (tarea 4.6 agregada), `.agents/memory/MEMORY.md` (esta entrada). Ningún archivo de código tocado.

**Para el siguiente agente:** si el usuario trae un `template_id` de HeyGen, retomar la tarea 4.6. La función nueva va en `video_provider.py` junto a `_submit_video`/`generate_spot_variations` — mismo patrón de polling que ya existe para `/v3/videos`, pero apuntando a `/v2/template/{id}/generate`. La categoría "noticias" del personaje (tarea 4.2, aún sin implementar) es el gancho natural para elegir qué `template_id` usar.

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

---

## [2026-07-17] — Agente Orquestador — Auditoría y corrección de Fase 002 + rediseño profesional del frontend

**Contexto:** el usuario pidió una revisión completa y honesta de lo ya implementado (backend + frontend) para convertirlo en un producto profesional para **Canal 11 TVU (UAGRM)**. Alcance acordado explícitamente: sin logo oficial (identidad textual propia, sustituible), y sin construir Fase 003 (Spots/video) — esa sigue pendiente tal como estaba.

**Bugs reales encontrados y corregidos (verificados ejecutando contra el Supabase real del proyecto, no solo leyendo código):**

1. **NSFW de salida nunca se ejecutaba.** `nsfw_filter.check_image_nsfw()` existía desde Fase 002 pero ningún caller la invocaba (confirmado por grep). Las variaciones generadas se mostraban al usuario sin pasar el filtro de salida que SOUL.md §6 exige como no negociable. Corregido: `check_generated_images_nsfw()` descarga cada variación y la valida, fail-closed, con un reintento ante 429 (necesario porque Pollinations gratis rate-limita).
2. **`PUT /admin/users/{id}/limits` era un endpoint fantasma.** No existía columna de límite configurable; el endpoint manipulaba `characters_used` hacia atrás asumiendo que el tope siempre era 2, así que subir el límite a 5 no cambiaba el comportamiento real de `characters.py` (seguía comparando contra `2` hardcodeado). Corregido con columnas reales `characters_limit_override`/`spots_limit_override` en `users` + `app/services/limits.py` (`get_effective_limits`) como única fuente de verdad.
3. **CRUD de usuarios duplicado** entre `auth.py` (`/api/v1/auth/users`) y `users.py` (`/api/v1/users`). `auth.py` ahora solo tiene `login`/`me`; `users.py` es el único CRUD admin. Verificado: `/api/v1/auth/users` → 404.
4. **Categorías desincronizadas:** `create_character` validaba contra una lista hardcodeada `["deportes","noticias","entretenimiento"]`, no contra `SpotCategory` — una categoría nueva creada por el admin nunca podía usarse. Corregido para validar contra la DB; se agregó `seed_default_categories()` idempotente al arranque (la tabla podía quedar vacía en un despliegue nuevo).
5. **`get_week_start()` duplicada** en `characters.py` y `admin.py` — unificada en `services/limits.py`.
6. **CORS hardcodeado a localhost** — ahora sale de `settings.CORS_ORIGINS`.
7. **`oxlint` documentado como "ya configurado" en `.agents/skills/frontend.md` pero no lo estaba** (ni la dependencia ni el script `lint` existían en `package.json`). Instalado y conectado de verdad.

**Migración de DB ejecutada (con confirmación explícita del usuario):** `ALTER TABLE users ADD COLUMN IF NOT EXISTS characters_limit_override INTEGER` y lo mismo para `spots_limit_override`, contra el Supabase real del proyecto — `create_all()` no altera tablas ya existentes, solo crea las que faltan.

**Frontend — rediseño profesional:**
- Identidad textual "TVU Studio" / "Canal 11 · UAGRM" (sin logo — decisión explícita del usuario) en Landing, Login, Sidebar, `index.html`.
- Paleta nueva (navy + acento ámbar) reemplazando el azul genérico de plantilla, escala tipográfica y sombras en tokens de `App.css`.
- `Dashboard.tsx` y `Admin.tsx` usaban `fetch` crudo con header manual en vez de `authFetch` — la expiración de sesión no se aplicaba ahí. Migrados ambos.
- Estilos inline eliminados en `Characters.tsx`, loading states unificados con el patrón `Loader2` existente.
- Mojibake corregido en `store.ts`.
- `GenerateSpot.tsx` (placeholder de Fase 003) rediseñado como estado "Próximamente" profesional, sin implementar el pipeline real.

**Verificación ejecutada (no solo "compila"):**
- Backend: venv creado, `requirements.txt` instalado, `uvicorn` levantado real contra Supabase. Probado: login, `/auth/users` (404 esperado), `/users` (funcional), override de límites (7/12 persiste y se refleja en lecturas posteriores), categoría inválida (400) y válida (201), creación de personaje con NSFW de salida real conectado (confirmado por rechazos reales en `security.log` — algunos por 429 de Pollinations bajo rate-limit de mis propias pruebas repetidas, no por contenido).
- Frontend: `tsc -b && vite build` limpio, `npx oxlint src` sin warnings, `npm run dev` levantado y el proxy `/api → :8000` verificado end-to-end con curl real.
- **No se tomaron capturas de navegador** (sin `chromium-cli` ni Playwright instalado en este entorno Windows) — verificación a nivel HTTP/build/lint, no visual. Queda pendiente que un humano confirme visualmente.

**Riesgo operativo descubierto (no introducido por este trabajo, pero ahora visible):** conectar el NSFW de salida obliga a esperar la generación real de las 3 imágenes antes de responder — antes el endpoint respondía casi instantáneo (solo construía URLs, la generación quedaba diferida al `<img>` del navegador). Ahora `POST /characters` puede tardar 30-90s+ bajo Pollinations gratis, más aún si el proveedor rate-limita (429). Es el costo correcto de cumplir SOUL.md §6, no un bug — si molesta en uso real, la solución es una API key de Pollinations o mover el chequeo a `BackgroundTasks`, no revertir el filtro.

**Para el siguiente agente:** Fase 002 queda auditada y con los bugs reales cerrados; Fase 003 sigue exactamente donde estaba (decisión explícita de alcance). Si se agregan columnas a modelos ya migrados en Supabase, recordar que `create_all()` no las agrega solo — hay que correr el `ALTER TABLE` a mano.

---

## [2026-07-17] — Agente Orquestador — Integración de logos oficiales y colores institucionales reales

**Contexto:** el usuario proveyó los logos reales (Canal 11 TVU y UAGRM) y los colores oficiales (blanco, rojo, azul), reemplazando el esquema textual/ámbar provisional de la sesión anterior.

**Hecho:**
- Los archivos venían en `frontend/src/public/` — corregido: esa carpeta NO es la estática de Vite (Vite solo sirve `frontend/public/`, sibling de `index.html`). Movidos a `frontend/public/logo-tvu.jpg` y `logo-uagrm.jpg`.
- Paleta de `App.css` actualizada: `--primary: #1a56db` (azul, muestreado del logo) y `--danger: #d92626` (rojo, mismo muestreo) — antes era un ámbar `#f2a900` inventado por no tener assets.
- Logos integrados en Sidebar, Login y Landing dentro de un chip blanco (`.brand-logo-chip`) porque son JPEG con fondo blanco — sin el chip se verían con un borde gris feo sobre el tema oscuro.
- Favicon actualizado a `logo-tvu.jpg`.

**Verificado:** `npm run build` limpio, `npx oxlint src` limpio, dev server sirviendo `/logo-tvu.jpg` y `/logo-uagrm.jpg` con 200.

**Para el siguiente agente:** si se agregan más assets de marca, van en `frontend/public/`, nunca en `frontend/src/public/`.

---

## [2026-07-17] — Fase 003 implementada — Generación de spots con Shotstack

**Contexto:** el usuario proveyó `SHOTSTACK_API_KEY_SANDBOX` y `SHOTSTACK_ENV=sandbox` y pidió implementar la generación de video de spots con Shotstack, reemplazando "Kling Omni via Luma API" (decisión previa del 2026-07-15 que nunca llegó a implementarse — `video_provider.py` seguía siendo un placeholder).

**Conflicto detectado y resuelto con el usuario antes de escribir código:** se leyó la documentación oficial de Shotstack (SOUL.md §9.1) y se confirmó que es un motor de **composición/render** (timeline en JSON sobre assets ya existentes), no un generador de IA — no sintetiza movimiento hiperrealista de un personaje a partir de una foto, que es lo que SOUL.md §1/§5 describen como la salida del producto. Se presentó el trade-off al usuario, que eligió explícitamente: **Shotstack solo, sin motor de IA generativa de por medio.** Efecto práctico: el "video" es la imagen ya aprobada del personaje con un efecto de cámara (Ken Burns: zoomIn/zoomOut/slideLeft por variación) + el guión superpuesto como subtítulo — no hay movimiento real del personaje ni lip-sync. Esto reduce el alcance de SOUL.md §1 ("salida: video del spot") frente a lo originalmente descrito; se documenta aquí porque SOUL.md es inmutable y no se edita, pero el propio usuario resolvió el conflicto (protocolo de SOUL.md: "el conflicto se anota en MEMORY.md para que un humano lo resuelva" — ya resuelto).

**Implementado:**
- `app/services/video_provider.py` reescrito: `build_spot_edit()` arma el JSON de Shotstack, `_submit_render`/`_poll_render` hablan con `POST/GET /render` (`x-api-key`, sandbox = `https://api.shotstack.io/edit/stage`, producción = `.../edit/v1`). `generate_spot_variations()` envía los 3 renders primero y los espera en paralelo (`asyncio.gather`) en vez de uno por uno, para no triplicar la espera.
- `app/config/settings.py`: `SHOTSTACK_API_KEY_SANDBOX`, `SHOTSTACK_API_KEY_PRODUCTION`, `SHOTSTACK_ENV`.
- `Spot` (modelo) ganó columna `variations_data` (Text, mismo patrón que `Character.consistency_data`) — **migrada a mano contra Supabase** (`ALTER TABLE spots ADD COLUMN IF NOT EXISTS variations_data TEXT`), con confirmación explícita del usuario antes de correrla.
- `app/api/v1/spots.py` (nuevo router, `/api/v1/spots`): `POST` (crea 3 variaciones; exige personaje `status=active`, valida guión 10-500 chars + NSFW de texto fail-closed, chequea límite semanal `VID_001`), `GET` (lista/detalle), `POST /{id}/select` (decrementa el límite solo al elegir, SOUL.md §5), `POST /{id}/redo` (máx 3, no decrementa).
- `frontend/src/pages/GenerateSpot.tsx`: implementado de punta a punta (antes era el placeholder "Próximamente"), mismo patrón visual que `Characters.tsx` (formulario → loading → grid de variaciones con `<video controls>` → seleccionar/rehacer → lista de spots existentes). CSS: se agregaron reglas `video` a `.variation-card` (16:9) y `.character-card-image` que solo tenían `img`.

**Verificado ejecutando (no solo compilando):** con confirmación explícita del usuario se corrió la migración contra Supabase real y se levantaron ambos servidores (ya estaban corriendo de una sesión previa con `--reload`, se confirmó que habían recargado el código nuevo vía `/openapi.json`). Contra el sandbox real de Shotstack: `POST /api/v1/spots` devolvió 3 URLs `.mp4` reales de S3 en ~14s (confirmado `Content-Type: video/mp4` real con `curl -I`); `select` decrementó `spots_used` (0→1) y persistió `output_url`; casos negativos probados y correctos: guión corto (400), tipo inválido (400), guión con palabra bloqueada (422 `GEN_004`, logueado en `security.log`), personaje no activo (400). Frontend: `tsc -b` y `npx oxlint src` limpios.

**Deuda/limitación descubierta, no resuelta:** las URLs de salida en modo `sandbox` incluyen `x-amz-expiration: Delete After 24 Hours` — los videos seleccionados por un usuario se van a caer del bucket de Shotstack al día siguiente. Para producción real hace falta la key de `SHOTSTACK_API_KEY_PRODUCTION` (hoy `xxxxx` en `.env`) y probablemente descargar/re-alojar el `output_url` en storage propio antes de las 24h si se quiere persistencia real. No se implementó descarga/rehosting — fuera de alcance de esta sesión, anotado para Fase 004.

**Para el siguiente agente:** el "video" de un spot es una composición, no una generación de IA — si en el futuro se quiere movimiento real del personaje, hace falta sumar un proveedor imagen-a-video (Kling/Luma/Runway/etc.) antes de Shotstack, no en su lugar. `EFFECTS` en `video_provider.py` es la lista de efectos de cámara por variación, fácil de ampliar.

---

## [2026-07-17] — Fase 003 — Se agregó narración (el usuario notó que los spots no tenían audio)

**Contexto:** justo después de la entrega de la sesión anterior, el usuario preguntó "¿por qué no tiene sonido y solo usa una imagen estática que se mueve?" — pregunta legítima: el spot original no llevaba ningún asset de audio. Pidió agregar sonido.

**Primer intento descartado (verificado en vivo, no asumido):** iba a reusar Pollinations (mismo proveedor que las imágenes, cero dependencias nuevas) con `text.pollinations.ai/{prompt}?model=openai-audio`. Al probarlo contra el endpoint real, devolvió `404 "Model not found: openai-audio... this is our legacy API"` — Pollinations deprecó el TTS anónimo gratuito; el reemplazo (`gen.pollinations.ai`, moneda "pollen") es de pago y requiere cuenta. Se descartó antes de escribir código de producción sobre un supuesto no verificado (SOUL.md §9.1).

**Solución real:** Shotstack tiene un asset nativo `type: "text-to-speech"` que se usa directo en el mismo timeline del Edit API — sin proveedor ni key adicional. Se usa voz **Lupe** (es-US) con `newscaster: true` (modo presentador de noticias; Shotstack solo soporta `newscaster` con las voces Matthew/Joanna en-US, Amy en-GB o Lupe es-US — Lupe es la única opción en español con ese modo).

**Cambio de diseño de duración:** se abandonó la idea de estimar la duración del habla por conteo de palabras (`WORDS_PER_SECOND`, llegó a implementarse y luego se descartó) — innecesaria porque ahora la narración vive dentro del mismo render de Shotstack. `DURATION_SECONDS = {"short": 5, "long": 30}` (tope superior de cada rango de SOUL.md §5) es ahora la longitud fija de los 3 clips (imagen, título, TTS). Si la narración real dura menos, sobra silencio sobre la imagen; si dura más, Shotstack la corta ahí — es el límite duro del producto, no un bug.

**Verificado ejecutando (no solo leyendo docs):** `POST /api/v1/spots` real contra el sandbox, se descargó uno de los 3 `.mp4` resultantes y se confirmó con una búsqueda binaria simple (sin `ffprobe` disponible en este entorno) que el archivo contiene los boxes `soun` (pista de audio), `mp4a` (códec AAC) y `vide` (pista de video) — no es un video mudo. `select` sigue funcionando igual.

**Para el siguiente agente:** no agregues un segundo proveedor de TTS — Shotstack ya lo resuelve nativo. Si se quiere cambiar de voz/idioma, la lista de voces con `newscaster` está en `video_provider.py` (comentario) y en la documentación de Shotstack (`generating-assets/ai-speech-generation`).

---

## [2026-07-17] — Fase 003 — Montage multi-plano (el usuario pidió avatar animado, decidió no pagar un proveedor todavía)

**Contexto:** el usuario pidió "avatar animado con movimiento que interactúe a lo largo del video" — es decir, movimiento real del personaje, no solo Ken Burns sobre una foto. Eso exige un proveedor de foto-a-avatar-hablante (categoría distinta a Shotstack: HeyGen, Hedra, D-ID, Akool). Se investigó en vivo (docs oficiales) y se presentó comparación al usuario — mejor encaje: **HeyGen** (su Photo Avatar no exige verificación de consentimiento para una cara no-real, entrada ~$5) y **Hedra** como alternativa (tier gratis 100 créditos, más enfocado a gesto/cuerpo). El usuario respondió **"Ninguno por ahora"** — no quiere sumar un proveedor de pago todavía.

**Lo que se hizo en su lugar (sin proveedor nuevo, solo exprimiendo Shotstack):** en vez de un único plano fijo con un efecto de cámara para todo el spot, `build_spot_edit()` ahora arma 2 planos (spots "short", 2.5s c/u) o 3 planos (spots "long", 10s c/u) de la MISMA foto aprobada del personaje — cada plano con su propio recorte (`ImageAsset.crop`, `SHOT_CROPS`) para simular un encuadre distinto, su propio efecto de cámara (`CAMERA_EFFECTS`, rotado por variación+segmento para que las 3 variaciones no se vean iguales entre sí) y fundido (`transition: fade`) entre planos. El subtítulo también se divide en `_script_chunks()` para que el texto cambie junto con cada plano en vez de mostrar el guión completo fijo. La narración TTS sigue siendo un solo clip de principio a fin (no tiene sentido cortarla).

**Sigue sin ser movimiento real del personaje** — es la misma técnica que un documental usa para narrar sobre una foto de archivo (varios "planos" de la misma imagen). Documentado explícitamente en el docstring de `video_provider.py` para que no se asuma erróneamente que esto ya resuelve el pedido original de "avatar animado".

**Verificado ejecutando:** render real contra el sandbox de un spot "long" (30s, 3 planos) — se descargó el `.mp4` resultante y se parseó el box `mvhd` del contenedor (sin `ffprobe` disponible en este entorno) para confirmar que la duración real es **30.0s exactos** (no que solo el primer segmento se renderizó y el resto se perdió) y que `mp4a`/`soun`/`vide` siguen presentes. `redo` y `select` probados sobre ese spot, ambos 200.

**Para el siguiente agente:** si en algún momento el usuario decide sumar un proveedor de avatar animado (HeyGen/Hedra), la investigación completa con endpoints/pricing/pros-contras ya quedó hecha en esta conversación — no la repitas desde cero, pero SÍ vuelve a verificar contra los docs oficiales en el momento porque este campo cambia rápido.

---

## [2026-07-17] — Agente Frontend — Overhaul visual: tema claro/oscuro + skeletons

**Hice:** tarea puramente visual/UX pedida por el usuario (sin tocar backend ni contrato de API). Dos ejes: (1) sistema de tema claro/oscuro real con persistencia y sin flash, (2) loading skeletons con la silueta real del contenido, reemplazando los spinners genéricos que solo se usaban para *fetch* de datos ya conocidos en forma (no para generación de IA).

**Archivos:**
- `frontend/index.html` — script inline en `<head>` que fija `data-theme` en `<html>` ANTES del primer paint (lee `localStorage["avatares-theme"]`, si no existe usa `prefers-color-scheme`). Evita flash de tema incorrecto.
- `frontend/src/store.ts` — agregado `useThemeStore` (zustand + `persist`, misma clave de patrón que `avatares-auth`), key `avatares-theme`. `toggleTheme`/`setTheme` escriben el atributo `data-theme` en `document.documentElement` directamente (no hay `useEffect` de sincronización porque el propio store ya es la única fuente de verdad después del primer paint).
- `frontend/src/App.css` — reescrito el bloque `:root` (tema oscuro = valores históricos, ahora documentado como tal) y agregado `:root[data-theme="light"]` con la paleta clara (mismo azul `#1a56db` / rojo `#d92626` institucional, fondos/superficies claros, `--success`/`--warning` oscurecidos en el tema claro porque se usan como color de texto plano sobre fondo claro y el tono original no pasa AA sobre blanco). Se corrigieron TODOS los `color: white` hardcodeados que debían seguir el tema (headings, `.card-value`, `.modal h3`, `.login-card h1`, `.sidebar-header h2`, `.landing-content h1`, `.landing-nav .brand-name`, `.feature h3`, `.character-card-info h3`, `.user-name`, `.limit-control span`) — los que quedaron en blanco literal son texto sobre fondo sólido de color (botones `.btn-primary`/`.btn-danger`/`.btn-select`, `.selected-badge`), esos SÍ deben quedar blancos en ambos temas. También se corrigieron varios `rgba(59, 130, 246, …)`/`rgba(239, 68, 68, …)` que usaban el azul/rojo genérico de Tailwind en vez del azul/rojo real de marca (`--primary`/`--danger`), y los hovers "tinte blanco" (`rgba(255,255,255,.05)`) que solo funcionaban sobre fondo oscuro pasaron a `color-mix(in srgb, var(--text) N%, transparent)` para funcionar en ambos temas sin variable nueva. Se agregó `color-scheme: dark`/`light` para que los controles nativos del navegador (scrollbar, etc.) también seleccionen su variante correcta.
- `frontend/src/components/ThemeToggle.tsx` (nuevo) — botón sol/luna (`lucide-react`, ya era dependencia) compartido por `DashboardLayout`, `Landing` y `Login` — los 3 puntos de entrada de la UI (dentro y fuera del layout autenticado).
- `frontend/src/components/Skeleton.tsx` (nuevo) — 3 componentes chicos reutilizando las clases reales (`.character-card`, `.card`, celdas de `.users-table`) más un shimmer CSS (`.skeleton`, `@keyframes skeleton-pulse`): `CharacterCardSkeleton` (usado en `Characters.tsx` y `GenerateSpot.tsx`, misma forma para lista de personajes y lista de spots), `StatCardSkeleton` (usado en `Dashboard.tsx` y `Admin.tsx`), `TableRowSkeleton` (usado en `Admin.tsx`).
- `frontend/src/pages/Characters.tsx` — el `loading` inicial (categorías) y `loadingCharacters` (lista) pasan de `inline-loading` con spinner a `CharacterCardSkeleton` × 3. El spinner por-imagen dentro de `.image-loading` (mientras carga una variación individual) pasa a shimmer (`className="image-loading skeleton"`) porque ahí también se conoce la forma exacta (cuadrada). Los spinners de "generando personaje" / "generando nuevas variaciones" (`.loading-overlay`, tiempo variable de IA) NO se tocaron — siguen siendo spinner+mensaje, que es lo honesto ahí.
- `frontend/src/pages/GenerateSpot.tsx` — mismo patrón: `loading` (personajes) y `loadingSpots` (lista) a `CharacterCardSkeleton` × 3 (reutiliza la misma clase `.character-card` que ya usaba la lista de spots). `.loading-overlay` de generación de spot/rehacer sin tocar.
- `frontend/src/pages/Dashboard.tsx` — antes NO tenía ningún estado de loading (mostraba 0/2 por defecto instantáneamente). Se agregó `const [loading, setLoading] = useState(true)` + `finally { setLoading(false) }` en `fetchLimits` (cambio mínimo de lógica, necesario para poder mostrar el skeleton pedido) y se muestran 2 `StatCardSkeleton` mientras carga.
- `frontend/src/pages/Admin.tsx` — el `loading` inicial (usuarios+métricas+categorías) pasaba por un `inline-loading` de página completa; ahora renderiza un esqueleto de página completa con la MISMA estructura final (3 `StatCardSkeleton` en `.dashboard-cards`, tabs estáticos, `<table>` con headers reales + 3 `TableRowSkeleton`).

**Decisión:** el pedido explícito listaba `sidebar-header h2` y `landing-nav brand` entre los que "deben seguir el tema", así que se optó por un flip COMPLETO de tema (sidebar incluido) en vez de un sidebar-oscuro-permanente (que había considerado como alternativa "con más personalidad" pero contradecía la instrucción explícita). No se agregó ninguna dependencia nueva — solo CSS variables + atributo `data-theme` + zustand (ya usado) + `color-mix()` nativo (soportado en navegadores modernos, no hace falta polyfill para este proyecto). Los skeletons son 3 componentes chicos reutilizados en 2-4 páginas cada uno (justifica la abstracción); el skeleton de imagen-de-variación individual se dejó inline (una sola clase CSS, un solo sitio de uso, no amerita componente).

**Verificado ejecutando:** `npx tsc -b` limpio, `npx oxlint src` limpio (exit 0), `npm run build` compila y genera `dist/` sin errores. Con el backend (`:8000`, `/docs` responde 200) y el frontend (`:5173`, ya corrían de una sesión previa del usuario) levantados, se confirmó por `curl` que `index.html` servido por Vite incluye el script de tema inline, y que **todos** los archivos tocados (9 páginas/componentes + `store.ts`) transforman con 200 vía el dev server de Vite (un error de sintaxis ahí da 500 con overlay) — o sea, ningún módulo rompe en tiempo de transformación. **No se tomaron capturas de pantalla ni se recorrió el flujo con un navegador real** — no hay herramienta de automatización de navegador disponible en este entorno; queda pendiente que el usuario abra `localhost:5173` y confirme visualmente el toggle de tema y los skeletons.

---

## [2026-07-17] — Agente Frontend — Segunda pasada de UI/UX: landing rediseñada, `user-select`, animaciones CSS, botones

**Pedido del usuario tras revisar la pasada anterior:** landing genérica sin personalidad, texto de "chrome" (botones, nav, badges, tabs) se resaltaba al arrastrar el cursor, cero animación salvo el shimmer/spin ya existentes, botones planos, y fricciones de UX puntuales en los flujos reales. Sin dependencias nuevas, sin tocar fetch/lógica de negocio, todo en `App.css` + JSX mínimo.

**1. Landing (`frontend/src/pages/Landing.tsx` + bloque grande de `App.css`) — reescrita completa:**
- Nav pasa de `position: absolute` (flotando sobre el hero) a `position: sticky` con fondo translúcido (`color-mix` + `backdrop-filter: blur`) y borde inferior — ya no se superpone al contenido, se comporta como header real.
- Hero pasa de una columna centrada con un solo CTA a un grid de 2 columnas: copy (eyebrow "Canal 11 · UAGRM", H1 con highlight en azul de marca, descripción, CTA con icono `ArrowRight`, lista de confianza con `ShieldCheck`/`RefreshCw` que refleja reglas reales de SOUL.md — NSFW automático y 3 rehacer gratis, no inventadas) + una tarjeta "mockup" decorativa construida 100% en CSS (avatar con gradiente azul/rojo institucional, líneas de texto simuladas, badge "Personaje activo") para dar personalidad visual sin depender de assets nuevos. `aria-hidden` porque es puramente decorativa.
- Sección nueva "Cómo funciona" con 3 pasos numerados que son el flujo real (crear → elegir variación → generar spot, SOUL.md §4-§5), con icon-circle (`.step-icon`) — refuerza a un visitante no autenticado qué va a hacer, no solo qué es el producto.
- Feature cards (ya existían) ahora con icono en círculo de color (`.feature-icon-wrap`) y hover-lift (`translateY(-4px)` + sombra), en vez de tarjetas planas sin interacción.
- Se eliminaron clases muertas (`.landing-content`, `.landing-subtitle`, `.landing-hero-logos`) que ya no se usan tras el rediseño — grep confirmó cero referencias antes de borrarlas.
- Mismo azul `#1a56db`/rojo `#d92626` y logos reales (`/logo-tvu.jpg`, `/logo-uagrm.jpg`) sin cambios — se elevó la identidad existente, no se inventó una nueva (instrucción explícita del usuario).

**2. `user-select: none` en el chrome, NO en contenido real (`App.css`, bloque nuevo tras el `body{}`):**
- Aplicado por selector explícito (no `* { user-select: none }` con excepciones después) a: todos los `button`/`a`, `.tab-btn`, `.theme-toggle`, `.sidebar-nav`, `.sidebar-user`, `.landing-nav`, `.landing-footer`, labels de formulario, `.page-header`/`.section-header` (cubre los `h1`/`h2` de título de página por herencia de `user-select`), `.admin-tabs`, `.batch-title`, `.card h3` (labels de stat card, ej. "Personajes esta semana" — NO el valor), títulos de modal, badges de estado/selección/límites.
- Restaurado explícitamente `user-select: text` en `input`, `textarea`, `select`, `.users-table td` (datos reales de usuarios: email, nombre), `.character-card-info` (nombre/categoría/descripción del personaje), `.profile-info` (email/nombre/rol del usuario en su propia página de perfil) — por si alguna regla más genérica los hubiera alcanzado por herencia.
- Caso encontrado y corregido en el camino: `.modal-subtitle` casi quedó en la lista de "no seleccionable" pero es donde Admin.tsx muestra el **email real** del usuario en el modal de límites (`{selectedUser.user_email}`) — se sacó de la lista antes de terminar, ese es exactamente el tipo de dato que el pedido dijo que debía seguir seleccionable.

**3. Animaciones CSS (sin librería, todo `transition`/`@keyframes`, guardadas con `prefers-reduced-motion`):**
- `.page` (wrapper compartido por TODAS las páginas del layout autenticado) anima `page-fade-in` (opacity+translateY) al montar — cubre el fade-in de contenido en cada cambio de ruta sin tocar cada página individualmente.
- `.card`, `.character-card`, `.feature` ganan hover-lift (`translateY` + sombra) consistente; `.variation-card` ya tenía cambio de borde en hover, se le sumó el mismo lift.
- Botones (`.btn-primary/.btn-secondary/.btn-danger/.btn-select/.btn-logout/.tab-btn`): sombra al hover, `scale(0.97)` al `:active` (feedback de "press"), `:focus-visible` con anillo (`box-shadow` con `--primary-soft`, antes dependían del outline por defecto del navegador nada más).
- `.modal-overlay`/`.modal`: antes aparecían instantáneos, ahora tienen animación de entrada (`modal-overlay-in` fade, `modal-in` fade+scale+translateY). Sin animación de salida porque React los desmonta condicionalmente, no hay fase que animar.
- Entrada escalonada (`card-fade-in` + `animation-delay` por `nth-child`, hasta el 6to hijo) en `.characters-list`, `.variations-grid` (grillas de personajes/variaciones/spots) y en `.landing-features`/`.landing-steps-grid`.
- Todo lo anterior está condicionado a `@media (prefers-reduced-motion: no-preference)` para que NO se dispare si el usuario pidió menos movimiento; además hay un bloque final `@media (prefers-reduced-motion: reduce)` que apaga explícitamente `transition`/`transform` en hover/active de tarjetas y botones (el movimiento disparado por interacción también cuenta). El shimmer de skeleton y el spinner de generación de IA (ya existentes, de la pasada anterior) NO se tocaron — siguen siendo la señal de "algo está pasando" incluso con reduced-motion, criterio común de accesibilidad para loading indicators.

**4. Botones (`.btn-primary`/`.btn-secondary`/`.btn-danger`/`.btn-select`/`.btn-logout`/`.tab-btn`):** además de hover/active/focus-visible del punto 3, se unificó `display: inline-flex; align-items: center; gap: 0.4rem` para que ícono+texto queden alineados igual en todos (antes dependía del layout inline por defecto de `<button>`, inconsistente entre botones con y sin ícono). Estados `:disabled` normalizados a `opacity: 0.5; cursor: not-allowed` + sin transform/sombra en todos, no solo en `.btn-primary` como antes.

**5. Fricciones de UX corregidas (sin tocar lógica/fetch):**
- Al elegir una variación (personaje o spot), las variaciones NO elegidas ahora se atenúan (`.variation-card.dimmed`, `opacity: 0.45` + `grayscale`) en vez de simplemente perder su botón sin ninguna señal visual de qué pasó — guía la vista hacia la elegida. Cambio de una línea de clase condicional en `Characters.tsx` y `GenerateSpot.tsx`, sin tocar el flujo de selección/rehacer.
- `Admin.tsx`: todos los botones de acción que antes eran solo texto ("Límites", "Eliminar", "Editar", "Crear Usuario", "Crear Categoría", "Guardar"/"Crear" y "Cancelar"/"Cerrar" en modales) ganaron ícono (`SlidersHorizontal`, `Trash2`, `Pencil`, `Plus`, `Check`, `X`) + `title` en los de solo-tabla, para consistencia con el resto de la app (Characters/GenerateSpot ya usaban ícono+texto en todos sus botones) y afordancia más clara.

**Verificado ejecutando:** `npx tsc -b` limpio, `npx oxlint src` limpio (exit 0), `npm run build` compila sin error. Con backend (`:8000`, `/docs` → 200) y frontend (`:5173`) ya corriendo, se confirmó por `curl` que los 5 archivos tocados (`Landing.tsx`, `App.css`, `Admin.tsx`, `Characters.tsx`, `GenerateSpot.tsx`) transforman con 200 vía el dev server de Vite. Se verificó balance de llaves de `App.css` (script en Node, profundidad final 0) tras todas las ediciones. **Sigue sin haber herramienta de automatización de navegador en este entorno** (mismo bloqueo que la pasada anterior) — no se tomaron capturas ni se recorrió el flujo clickeando en un navegador real; pendiente que alguien con navegador abra `localhost:5173` y confirme visualmente landing, tema claro (se revisó que las clases nuevas usan `var(--...)` existentes, ninguna con color fijo fuera de los ya establecidos como "texto blanco sobre fondo sólido"), y que el `user-select` no rompió la selección de texto en inputs/tablas/nombres de personaje.

**Para el siguiente agente:** si agregás una página nueva con lista de datos fetcheados, reusá `CharacterCardSkeleton`/`StatCardSkeleton`/`TableRowSkeleton` de `frontend/src/components/Skeleton.tsx` en vez de un `inline-loading` con spinner — ese patrón queda reservado para casos donde NO se conoce la forma final (o para generación de IA con tiempo variable, ahí seguí usando `.loading-overlay`). El toggle de tema vive en `useThemeStore` (`frontend/src/store.ts`) — no dupliques la lógica de persistencia, usá `useThemeStore((s) => s.theme)` / `toggleTheme()`.

---

## [2026-07-18] — Agente Frontend — Tercera pasada UI/UX: dashboard responsive + Fitts + jerarquía de CTA (VERIFICADA EN NAVEGADOR)

**Hice:** intervención quirúrgica de UX/UI (alcance "roto + refinamiento visual" elegido por el usuario), anclada a leyes de UX que pidió justificar (Fitts, Hick). NO fue rediseño total — se rechazó explícitamente porque tiraba dos rondas de trabajo válido.

**Cambios:**
1. **Dashboard responsive (el hueco grave):** en 1806 líneas de CSS el único `@media (max-width)` era `860px` y solo cubría la landing; el shell (`DashboardLayout`) era `sidebar` 240px fijos + `main` en flex, sin colapso — en mobile la barra se comía la pantalla. Ahora, debajo de 900px: topbar sticky con hamburguesa, sidebar off-canvas (`.sidebar.open` + `translateX`), backdrop que cierra al tocar fuera, botón X, y cierre automático al navegar (`useEffect` sobre `location.pathname`). En desktop (≥900px) NADA cambió. Transición del off-canvas apagada en `prefers-reduced-motion: reduce`.
2. **Ley de Fitts:** `.theme-toggle` 36→44px (mínimo táctil); `.btn-secondary`/`.btn-danger` con `min-height: 2.5rem` (40px) — los botones de tabla de Admin tenían ~30px.
3. **Ley de Hick / jerarquía:** el hero de la landing repetía el mismo botón del nav; ahora tiene primario "Iniciar Sesión" + secundario "Cómo funciona" (ancla `#como-funciona` con `scroll-behavior: smooth` guardado por reduced-motion).

**Archivos:** `frontend/src/components/DashboardLayout.tsx` (topbar+hamburguesa+estado `menuOpen`), `frontend/src/App.css` (bloque nuevo al final: Fitts, landing CTA, shell responsive, smooth-scroll; + edit del `.theme-toggle`), `frontend/src/pages/Landing.tsx` (CTA secundario + `id` de sección).

**Decisión:** bisturí, no excavadora. El frontend ya tenía dos rondas de UI/UX (tokens, tema claro/oscuro, skeletons, `focus-visible`, `prefers-reduced-motion`) — rehacerlo habría sido churn destructivo. Se tocaron 3 archivos, cero dependencias, cero cambios de API/lógica.

**Verificado ejecutando + POR FIN EN NAVEGADOR:** `tsc -b` limpio, `oxlint src` exit 0, dev server responde 200. **El usuario abrió `localhost:5173` y confirmó visualmente** ("Está bien el frontend") — cierra el punto "recorrer el flujo en el navegador" del DoD de frontend.md que quedó pendiente las DOS pasadas anteriores (2026-07-17).

**Para el siguiente agente:** si agregás items al sidebar o botones al chrome, el patrón mobile ya está — el sidebar off-canvas y la topbar viven en `DashboardLayout.tsx`, el CSS responsive en el bloque final de `App.css` bajo `@media (max-width: 900px)`. El breakpoint del shell es 900px; el de la landing sigue en 860px (no los unifiqué, son componentes distintos). Cualquier botón nuevo hereda automáticamente el min-height de Fitts si usa `.btn-secondary`/`.btn-danger`.

---

## [2026-07-20] — Agente Frontend — Formulario de creación de personaje: campos guiados de apariencia/estilo

**Pedido del usuario:** mejorar el formulario de creación de personaje para que sea más personalizable y preciso, considerando que los avatares se usan tanto para spots de noticias como para creación de contenido de valor.

**Hice (solo `frontend/src/pages/Characters.tsx`, cero cambios de backend/DB):** el campo `description` ya viaja sin cambios a `build_character_prompt()` (`backend/app/services/image_provider.py`), así que la precisión se ganó agregando selects guiados en vez de dejar solo un textarea libre — el usuario rara vez completa un textarea vacío con suficiente detalle para un prompt de generación de imagen.

- 6 selects opcionales nuevos: género, edad aparente, tono de piel, cabello, vestimenta, y **estilo de presentación** (este último mapea directo al caso de uso: "formal como presentador de noticias" vs "cercano como creador de contenido" vs enérgico/serio) — todos como constantes `_OPTIONS` arriba del componente.
- `composeDescription()` combina las selecciones + el textarea libre (renombrado "Detalles adicionales") en una sola oración, recortada a 500 caracteres (límite ya validado en backend, `MAX_DESCRIPTION_LENGTH`).
- Vista previa en vivo de la descripción compuesta antes de enviar (transparencia sobre qué prompt se va a usar).
- Validación nueva: si no hay imagen de referencia NI descripción compuesta, bloquea el submit — antes el form dejaba crear un personaje sin imagen y sin texto (violaba SOUL.md §1 "al menos uno de los dos es obligatorio", que el backend tampoco valida — **deuda anotada abajo**, no corregida en backend porque el pedido era del formulario).

**Verificado ejecutando:** `npx tsc -b` limpio, `npx oxlint src` exit 0, `npm run build` compila sin error. **No se recorrió en navegador real** (sin herramienta de automatización en este entorno, mismo bloqueo recurrente) — pendiente confirmación visual humana de los 6 selects nuevos y la vista previa.

**Deuda técnica nueva:** el backend (`POST /api/v1/characters`) NO valida que exista imagen o descripción — solo el frontend lo exige ahora. Si se llama al endpoint directo (Swagger, script), se puede crear un personaje sin ninguno de los dos, violando SOUL.md §1. No se corrigió porque el pedido era del formulario, no del endpoint — anotado en backlog.md.

**Para el siguiente agente:** si agregás otro campo guiado, sumalo a `composeDescription()` en `Characters.tsx` — no dupliques la construcción del prompt en otro lado, el backend sigue recibiendo un solo string `description`. `GenerateSpot.tsx` no se tocó — reutiliza el `reference_image_url` ya persistido, no pasa por este flujo de descripción.

---

## [2026-07-20] — Agente Backend+Frontend — Migración de Shotstack a HeyGen (avatar real animado) + selector de voz

**Pedido del usuario:** consiguió API key paga de HeyGen (motivo original del bloqueo de Fase 003: Shotstack es composición, no anima al personaje). Pidió reemplazar el proveedor de video, luego limitar a 1 variación mientras prueba (costo real por video), y después agregar selector de voz con preview por personaje.

**Hice:**
1. **`backend/app/services/video_provider.py` reescrito completo** — ya no usa Shotstack, usa HeyGen API v3 `POST /v3/videos` con `type: "image"` (anima la MISMA `reference_image_url` del personaje con lip-sync real sobre el guión exacto). Se investigó en vivo contra `developers.heygen.com` (SOUL.md §9.1) antes de implementar: se descartó a propósito el "Video Agent" (`/v3/video-agents`) porque ese trata el guión como concepto a elaborar creativamente (agrega escenas, reescribe texto) — no sirve para un spot con mensaje exacto aprobado por el usuario. `/v3/videos type=image` hace TTS verbatim del campo `script`, sin liberties creativas.
2. **`_resolve_voice_id`** auto-descubre la primera voz en español del catálogo de HeyGen (cacheada en memoria por proceso) cuando el personaje no tiene voz elegida — fallback para personajes viejos.
3. **`list_spanish_voices()`** nueva — catálogo de voces (voice_id, name, gender, preview_audio_url) para el selector del frontend.
4. **Settings nuevos** (`backend/app/config/settings.py`): `HEYGEN_API_KEY`, `HEYGEN_VOICE_ID` (override manual opcional), `HEYGEN_SPOT_VARIATIONS` (default 3 = SOUL.md §5; en `.env` está en `1` temporalmente mientras el usuario prueba, para no gastar créditos — **subir a 3 cuando confirme que funciona**, ver `backlog.md`).
5. **Modelo `Character`** — 2 columnas nuevas: `heygen_voice_id`, `heygen_voice_name` (voz elegida por personaje, misma idea que `reference_image_url`: consistencia entre todos los spots del personaje). **Migradas a mano contra Supabase real** (`ALTER TABLE characters ADD COLUMN IF NOT EXISTS ...`), mismo patrón que `variations_data` — `create_all()` no las hubiera creado solo.
6. **Endpoints nuevos en `characters.py`:** `GET /api/v1/characters/heygen-voices` (catálogo, cualquier usuario autenticado) y `PATCH /api/v1/characters/{id}/voice` (fija `heygen_voice_id`/`heygen_voice_name`, valida ownership). Declarados ANTES de `GET /{character_id}` en el router — si no, FastAPI matchea `"heygen-voices"` como si fuera un `character_id`.
7. **`spots.py`** — ambos endpoints (`create`/`redo`) pasan `character.heygen_voice_id` a `generate_spot_variations` y usan `settings.HEYGEN_SPOT_VARIATIONS` en vez de `3` fijo.
8. **Frontend (`Characters.tsx`)** — botón "Voz"/nombre de voz en cada character-card activa → modal (`.modal-voices`) con lista de voces, cada una con `<audio controls>` nativo para preview (sin librería nueva) y botón "Usar esta voz" que llama al PATCH y actualiza la card en memoria.

**Archivos:** `backend/app/services/video_provider.py` (reescrito), `backend/app/config/settings.py`, `backend/app/models/models.py`, `backend/app/schemas/character.py`, `backend/app/api/v1/characters.py`, `backend/app/api/v1/spots.py`, `backend/.env`, `frontend/src/pages/Characters.tsx`, `frontend/src/App.css`.

**Decisión:** cambio de proveedor completo, no A/B — el usuario compró HeyGen específicamente para resolver el bloqueo documentado en HEARTBEAT.md ("Ninguno por ahora" del 2026-07-17 queda revertido). Se eliminó todo el código de Shotstack de `video_provider.py` (composición de planos/crops/fundidos ya no aplica); `settings.py` conserva las variables `SHOTSTACK_*` sin usar por si se quiere volver atrás, no se borraron por ser bajo costo de mantenerlas.

**Verificado ejecutando (real, no solo compila):**
- Auth + auto-descubrimiento de voz contra la API real de HeyGen (sin costo, endpoint de solo lectura) — devolvió `voice_id` válido.
- `GET /api/v1/characters/heygen-voices` real → devuelve catálogo con `preview_audio_url` reales.
- `PATCH /api/v1/characters/{id}/voice` real contra un personaje activo existente en Supabase → `GET` posterior confirma persistencia.
- Negativos: sin token → 401, personaje inexistente → 404.
- `npx tsc -b` limpio, `npx oxlint src` exit 0.
- **No se generó ningún video de prueba real** (`POST /spots`) — consume créditos pagados de HeyGen; el usuario prefirió probarlo él mismo desde el frontend.

**Para el siguiente agente:**
- `HEYGEN_SPOT_VARIATIONS=1` en `.env` es temporal — si el usuario confirma que el flujo funciona, subirlo a `3` (o borrar la línea, el default en `settings.py` ya es `3`) para volver a cumplir SOUL.md §5. El backend necesita reiniciarse para tomar cambios de `.env` (pydantic-settings lo lee solo al arrancar).
- HeyGen tarda más que Shotstack (minutos, no segundos) — `_poll_video` tiene timeout de 900s (15 min) por video, ajustar si en la práctica no alcanza.
- Si se agrega un selector de avatar del catálogo de HeyGen (no solo voz): NO hacerlo — competiría con el flujo de creación de personaje propio del producto (SOUL.md §1/§4). Ya se decidió explícitamente no ofrecerlo, ver conversación con el usuario.

---

## [2026-07-20] — Agente Backend — Fix: preview de voz no sonaba (Content-Type roto del CDN de HeyGen)

**Reporte del usuario:** "no me deja escuchar el preview de las voces" en el selector agregado en la entrada anterior.

**Diagnóstico (verificado con `curl -I` real, no supuesto):** los `preview_audio_url` de `GET /v3/voices` son siempre MP3 real (magic bytes `ID3`, confirmado con `xxd`) sin importar la extensión de la URL (algunas terminan en `.wav`), pero el CDN de HeyGen a veces responde `Content-Type: binary/octet-stream` en vez de `audio/mpeg` para el mismo tipo de archivo. El `<audio src>` del navegador no reproduce un recurso si el `Content-Type` no es de audio — de ahí que algunas voces sonaran y otras no (dependía de qué endpoint del CDN sirviera esa voz en particular).

**Hice:** `video_provider.fetch_voice_preview(url)` — descarga el preview server-side y `characters.py` lo re-sirve en `GET /api/v1/characters/voice-preview?url=...` forzando `Content-Type: audio/mpeg` siempre. El frontend (`Characters.tsx`) ahora apunta el `<audio src>` a ese proxy en vez de la URL directa de HeyGen. Sin auth a propósito (son clips públicos del catálogo, no datos de usuario) para que el `<audio>` nativo no necesite manejar el header `Authorization`; para no ser un proxy abierto, `fetch_voice_preview` valida que el host termine en `.heygen.ai` antes de pedir la URL (400 si no).

**Archivos:** `backend/app/services/video_provider.py`, `backend/app/api/v1/characters.py`, `frontend/src/pages/Characters.tsx`.

**Verificado ejecutando:** `curl` real contra el proxy para las dos variantes (una que ya venía `audio/mpeg`, otra que venía `binary/octet-stream` en origen) — ambas responden `200` + `Content-Type: audio/mpeg` + los mismos bytes `ID3` del original. SSRF: URL de host no-HeyGen → `400`. `tsc -b`/`oxlint` limpios.

**Para el siguiente agente:** si se agrega cualquier otro proxy de recurso externo, seguir este patrón (validar host allowlist, forzar Content-Type conocido) en vez de confiar en lo que devuelve el proveedor externo.

---

## [2026-07-20] — Agente Backend+Frontend (modo plan) — Pollinations → HeyGen también en la CREACIÓN de personajes + se saca el bloqueo por límites

**Pedido del usuario:** "no sería bueno ver si hay alguna forma de mejorar la personalización... seleccionar el avatar hechos en HeyGen" — quería poder elegir avatares del catálogo de HeyGen Y crearlos con HeyGen (foto propia). Esto **revierte** la entrada de esta misma sesión donde se había descartado explícitamente un selector de avatar de HeyGen "para no competir con el flujo propio del producto" — el usuario cambió de opinión y lo pidió directo, así que no es un error del agente, es una decisión de negocio nueva.

**Por qué se usó modo plan:** la tarea tocaba SOUL.md (reglas de negocio), un cambio de DB, y removía una feature que funcionaba (Pollinations) — se armó un plan (`EnterPlanMode`/`ExitPlanMode`, guardado en `C:\Users\HP VICTUS\.claude\plans\eager-floating-river.md`) y se usó `AskUserQuestion` tres veces antes de escribirlo:
1. Alcance: ¿reemplaza Pollinations del todo o queda como alternativa? → **reemplazo total**.
2. Modos de creación con HeyGen habilitados → **foto propia + catálogo** (NO texto/prompt).
3. Mientras se armaba el plan, surgió que HeyGen devuelve UN solo resultado por foto (no 3 variaciones como Pollinations) y cuesta crédito real — se preguntó cómo debía contar eso contra el límite semanal, y la respuesta del usuario fue mucho más amplia de lo esperado: **"No es necesario los límites, [...] es una web para uso del canal 11 TVU de la UAGRM"** → esto se convirtió en un segundo cambio (sacar el bloqueo por límites) confirmado con una pregunta de alcance aparte: se eligió **"solo dejar de bloquear"** (se mantienen los contadores como métrica para el admin, no se borra la tabla `CharacterLimit` ni la UI de límites de `Admin.tsx`).

**Hice:**

1. **Modelo `Character`** — 2 columnas nuevas: `heygen_avatar_id` (look id, se usa como `avatar_id` en `POST /v3/videos`), `heygen_avatar_group_id`. Migradas a mano contra Supabase real (confirmado con `information_schema.columns`).

2. **`video_provider.py`:**
   - `create_avatar_from_photo(name, image_bytes, media_type)` — `POST /v3/avatars` type=photo con `file.type=base64` (evita necesitar hosting propio de assets), luego poll vía `GET /v3/avatars/looks?group_id=...&ownership=private` hasta `status=="completed"` (reusa el mismo endpoint que ya se usaba para el catálogo, en vez de sumar un endpoint de "get look" nuevo).
   - `list_public_avatar_looks(limit, token)` — proxy de `GET /v3/avatars/looks?ownership=public`, paginado con `next_token`.
   - `_submit_video`/`generate_spot_variations` ahora reciben `avatar_id` opcional: si está seteado arma `type: "avatar"`, si no cae a `type: "image"` con `reference_image_url` — **compatibilidad hacia atrás con los 3 personajes ya existentes** (Maria Noticias, Pepe la mosca x2) creados antes de este cambio, que solo tienen imagen de Pollinations.

3. **`image_provider.py` — borrado.** Ya no se genera nada por texto/Pollinations. Confirmado con grep que nada más lo importaba antes de borrarlo.

4. **`characters.py` — reescrito:**
   - Se eliminaron `POST /characters` (Pollinations, 3 variaciones), `POST /{id}/select`, `POST /{id}/redo` — el modelo "3 variaciones + rehacer" no aplica a HeyGen (determinístico, un solo resultado, cuesta crédito real cada intento).
   - `POST /create-from-photo` — sube foto, valida NSFW de entrada Y salida (mismo patrón que ya existía para Pollinations, reusando `check_image_bytes_nsfw`/`check_image_url_nsfw`), llama a HeyGen. **No persiste en DB** — devuelve el resultado para que el frontend muestre "¿usar este?" antes de gastar el slot. Si el usuario descarta, no queda ningún rastro en la base (el avatar sigue existiendo huérfano del lado de HeyGen, aceptado como costo menor — no hay endpoint de borrado de personajes todavía de cualquier forma, ver backlog 4.1).
   - `GET /heygen-catalog` — proxy del catálogo público, sin costo.
   - `POST /confirm` — acá recién se crea el `Character` (`status="active"` directo, sin "draft" intermedio porque no hay nada más que elegir) y se cuenta contra `characters_used` (métrica, ya no bloquea).
   - Se sacó el 403 `CHAR_001` que bloqueaba por límite semanal.

5. **`spots.py`** — se sacó el 403 `VID_001`. El gate de personaje activo ahora acepta `heygen_avatar_id` **o** `reference_image_url` (compat con personajes viejos). Se pasa `character.heygen_avatar_id` a `generate_spot_variations`.

6. **`schemas/character.py`** — se sacaron los schemas del flujo de variaciones (`CharacterCreate`, `CharacterVariation`, `CharacterCreateResponse`, `CharacterRedoResponse`, `CharacterSelectRequest`); se agregaron `CharacterFromPhotoResponse`, `HeygenCatalogAvatar`/`HeygenCatalogResponse`, `CharacterConfirmRequest`.

7. **`SOUL.md`** — reescrito §1 (entrada válida: foto propia o catálogo, ya no texto), §3 (límites pasan de bloqueo duro a métrica, con la razón: uso interno de Canal 11 TVU), §4 (flujo de creación completo, dos caminos, sin 3 variaciones/rehacer para personajes — el rehacer de SPOTS en §5 no cambió).

8. **Frontend `Characters.tsx`** — reescritura grande: se sacaron los 6 selects guiados + textarea de descripción + `composeDescription()` + toda la grilla/batches/rehacer de variaciones + el gating `canCreate`/banner de límite. Formulario nuevo: elegir modo ("Subir mi foto" / "Elegir del catálogo") → foto propia hace `create-from-photo` y muestra un único resultado con confirmar/descartar; catálogo pagina con `next_token` y confirma directo al hacer click en una tarjeta. El selector de voz (de la pasada anterior) no se tocó.

9. **Frontend `GenerateSpot.tsx`** — cambio puntual: se sacó el `canCreate`/banner de límite de spots (ya no bloquea). El resto (flujo de variaciones/rehacer de SPOTS) no cambió — eso sigue siendo válido, es sobre el VIDEO no sobre la creación del personaje.

**Archivos:** `backend/app/models/models.py`, `backend/app/services/video_provider.py`, `backend/app/services/image_provider.py` (borrado), `backend/app/api/v1/characters.py`, `backend/app/api/v1/spots.py`, `backend/app/schemas/character.py`, `.agents/memory/SOUL.md`, `frontend/src/pages/Characters.tsx`, `frontend/src/pages/GenerateSpot.tsx`, `frontend/src/App.css` (nuevo `.creation-mode-choice`/`.creation-mode-card`, se borró `.limits-badge` sin uso).

**Verificado ejecutando (real):**
- `python -c "import app.main"` limpio en cada paso.
- `GET /characters/heygen-catalog` real contra HeyGen → devolvió looks públicos reales (ej. avatar "Marco").
- `POST /characters/confirm` real con un look de ese catálogo → `GET /characters/{id}` confirmó persistencia con `heygen_avatar_id` seteado. Fila de prueba borrada de Supabase después (no hay endpoint de borrado en el producto, se hizo con SQL directo).
- `npx tsc -b`, `npx oxlint src`, `npm run build` limpios. `curl` a los archivos tocados vía el dev server de Vite → 200.
- **NO se probó `create-from-photo` ni `POST /spots` con un personaje creado por este flujo** — ambos consumen crédito real de HeyGen; el usuario dijo que los prueba él mismo desde el frontend (mismo patrón que ya se estableció en esta sesión para la generación de video).

**Para el siguiente agente:**
- Si un personaje viejo (sin `heygen_avatar_id`) todavía tiene `reference_image_url` de Pollinations, sigue funcionando para generar spots (`type: "image"`) — no hace falta migrar esos 3 personajes existentes.
- No hay endpoint para eliminar/editar personajes (backlog 4.1) — ahora es más urgente: un catálogo mal elegido o una foto confirmada por error queda pegada en la lista para siempre, sin forma de sacarla desde la UI.
- Si se agrega el modo "desde texto/prompt" con HeyGen más adelante (quedó explícitamente fuera de alcance esta vez), HeyGen sí soporta `type: "prompt"` en `POST /v3/avatars` — no investigado en profundidad en esta sesión, verificar en vivo antes de implementar.

---

## [2026-07-20] — Agente Backend+Frontend — Refresh token real (causa raíz de deslogueos durante la generación de spots)

**Reporte del usuario:** "por qué cuando genere el video de repente me expulsó y me cerró la sesión".

**Diagnóstico:** `AuthChecker` en `frontend/src/App.tsx` corre un `setInterval` cada 30s en toda página protegida y llamaba a `isAuthenticated()`, que miraba únicamente el `access_token` (`ACCESS_TOKEN_EXPIRE_MINUTES=60` en `settings.py`, **sin refresh token** — pese a que SOUL.md §9.6 ya decía "access corto + refresh 7 días", nunca se implementó). Como la generación de un video de HeyGen puede tardar varios minutos y el usuario venía de una sesión larga probando el flujo nuevo, el access token venció literalmente a mitad de la espera y el interval de 30s deslogueó de inmediato — sin relación causal directa con la generación en sí, solo coincidencia de timing por sesión larga.

**Pregunté** si prefería el parche rápido (subir el tiempo de expiración) o implementar el refresh token real que SOUL.md ya pedía — **eligió refresh token real**.

**Hice:**
- `settings.py` — `ACCESS_TOKEN_EXPIRE_MINUTES` 60→**30** (ahora sí coincide con SOUL.md), nuevo `REFRESH_TOKEN_EXPIRE_DAYS=7`.
- `auth_handler.py` — `create_access_token`/nuevo `create_refresh_token` agregan un claim `"type": "access"|"refresh"` para que uno no sirva como el otro. `decode_access_token` renombrado a `decode_token` (ahora genérico, decodifica cualquiera de los dos — el chequeo de tipo lo hace quien lo llama). Refresh token **no rotativo** (mismo token sirve hasta sus 7 días) — ponytail: rotación con storage/revocación en DB sería más seguro pero es un sistema interno de un solo canal, no expuesto públicamente; no se justificaba la complejidad.
- `dependencies.py` (`get_current_user`) — rechaza un token con `type != "access"` (bloquea usar el refresh token directo contra la API).
- `auth.py` — `POST /login` ahora devuelve `refresh_token` además de `access_token`. Nuevo `POST /auth/refresh` (body `{refresh_token}`) → valida tipo+firma+vigencia, devuelve un `access_token` nuevo.
- Frontend `store.ts` — `AuthState` gana `refreshToken`/`refreshTokenExpiry` y `setAccessToken()`. **`isAuthenticated()` cambia de significado:** antes miraba `tokenExpiry` (access), ahora mira `refreshTokenExpiry` — es la definición correcta de "hay sesión viva o no". Nueva `isAccessTokenExpired()` para que `authFetch` sepa cuándo renovar.
- Frontend `lib/api.ts` (`authFetch`) — reescrito: si el access token venció, llama a `/auth/refresh` solo (con dedup de refrescos concurrentes vía una promesa compartida) y reintenta, en vez de deslogueear. Si el refresh falla (o el refresh token también venció), recién ahí fuerza logout. También reintenta una vez tras un 401 inesperado del servidor antes de rendirse.
- `Login.tsx` — captura `refresh_token` de la respuesta y lo pasa a `setAuth`.
- `App.tsx` (`AuthChecker`) — **no se tocó**: como `isAuthenticated()` ya cambió de significado en el store, el mismo interval de 30s ahora hace lo correcto automáticamente (solo desloguea cuando el refresh de 7 días vence).

**Archivos:** `backend/app/config/settings.py`, `backend/app/auth/auth_handler.py`, `backend/app/auth/dependencies.py`, `backend/app/schemas/auth.py`, `backend/app/api/v1/auth.py`, `frontend/src/store.ts`, `frontend/src/lib/api.ts`, `frontend/src/pages/Login.tsx`.

**Verificado ejecutando (real):** login devuelve ambos tokens con su claim `type` correcto; `GET /characters` con access token → 200, con refresh token → 401; `POST /auth/refresh` con refresh token real → access token nuevo y funcional (200 en `/characters`); `POST /auth/refresh` con un access token (tipo incorrecto) → 401. `tsc -b`/`oxlint`/`build` limpios. (Nota aparte: durante la verificación el reload de `uvicorn --reload` tardó ~2s en tomar los cambios de `auth.py`/`schemas/auth.py` — un curl disparado justo en esa ventana dio una respuesta con el schema viejo y otro dio `000`; no es un bug del código, es una condición de carrera de mi propio script de verificación contra el autoreload, se resolvió solo reintentando 2s después.)

**Para el siguiente agente:**
- Cualquier sesión de navegador abierta ANTES de este cambio no tiene `refresh_token` en su `localStorage` persistido (`avatares-auth`) — la primera vez que su access token viejo venza, `isAuthenticated()` va a dar `false` (no hay refresh token) y va a desloguear una vez más. Después de volver a loguearse queda con el flujo nuevo y no debería cortarse de nuevo. Avisado al usuario.
- Si se agrega un endpoint de logout-en-servidor o revocación de sesiones más adelante, hoy no existe — el refresh token no rotativo vive sus 7 días completos sin forma de invalidarlo antes (aceptado como límite conocido, ver "Hice" arriba).
