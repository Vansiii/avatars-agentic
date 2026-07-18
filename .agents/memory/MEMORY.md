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
