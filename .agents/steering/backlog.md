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

- [x] **3.1** 🔴 · Endpoint POST /spots (personaje + script + tipo)
- [x] **3.2** 🔴 · Pipeline de generación de video — **Shotstack** (composición, reemplaza a Kling/Luma que nunca se implementó; decisión explícita del usuario 2026-07-17, ver MEMORY.md)
- [x] **3.3** 🔴 · Consistencia: personaje aparece en el video (usa reference_image_url como imagen base del composite)
- [x] **3.4** 🟡 · Tipos de video: short (4s) y long (20s)
- [x] **3.5** 🟡 · Selección de variación de video (3 opciones)
- [x] **3.6** 🟡 · Sistema de rehacer para spots (máx 3 veces)
- [x] **3.7** 🔴 · Control de límites semanales (5 spots/usuario/semana)
- [x] **3.8** 🟡 · Frontend: página de generar spot con flujo completo

---

## Fase 004 — Cierre del Alpha 🟢

- [ ] **4.1** 🟡 · Gestión de personajes: editar, eliminar, reasignar categoría
- [ ] **4.2** 🟡 · Categorías de spots predefinidas (deportes, noticias, etc.)
- [ ] **4.3** 🟢 · Dashboard con métricas de uso (admin)
- [ ] **4.4** 🟢 · Tests E2E del flujo completo
- [ ] **4.5** 🟢 · README de arranque: cómo levantar el sistema
- [ ] **4.6** 🟡 · Formato "noticiero" para spots de categoría noticias — usar el **Template API de HeyGen** (`POST /v2/template/{template_id}/generate`) en vez del endpoint plano `/v3/videos` que usa hoy `video_provider.py`. Requiere: (1) el usuario arma manualmente 1-2 plantillas en el editor web de HeyGen (fondo de estudio, rótulo inferior con nombre/cargo, logo de Canal 11 TVU, cintillo — HeyGen ya ofrece esto como producto "AI News Generator"); (2) el agente agrega una función en `video_provider.py` que llame ese endpoint mapeando `avatar_id`→variable `character`, guión→variable `text`/voz, nombre del personaje→variable del rótulo, y opcionalmente una imagen/video de B-roll; (3) `template_id` fijo por categoría (constante en `settings.py` o campo en `SpotCategory` si ya existe esa tabla — ver 4.2, encajan). De regalo: la Template API soporta `caption`/`subtitles` para subtítulos automáticos, útil para accesibilidad y recortes de redes. Bloqueado hasta que el usuario provea el `template_id` (no se puede probar sin costo real de HeyGen). NO usar Interactive/Streaming Avatar (HeyGen) para esto — es para avatares en vivo vía WebRTC, no para spots pregrabados, y su infra de sesión en tiempo real está fuera del alcance del Alpha (SOUL.md §8).

---

## Deuda técnica conocida

- Consistencia de personaje por imagen de referencia (no modelos avanzados) — suficiente para Alpha
- ~~Proveedor de video por definir (research pendiente)~~ ✅ Kling Omni via Luma API (2026-07-15)
- `create_all` en arranque en vez de Alembic — aceptable para Alpha, pero implica migrar columnas nuevas a mano contra Supabase (ver MEMORY.md 2026-07-17)
- NSFW filter: implementación básica con palabras clave + heurística de tamaño (Alpha) — en producción usar NudeNet o similar. Desde 2026-07-17 la salida SÍ se valida (antes no se llamaba); sigue siendo heurístico, no un modelo real.
- Eliminación de imágenes a 24h y strip de EXIF — pendiente para Fase 004
- **(2026-07-17) Latencia de `POST /characters`:** validar la salida obliga a esperar la generación real de las 3 imágenes antes de responder — con Pollinations gratis (sin API key) puede tardar 30-90s+ o más si rate-limita (429). Considerar API key de Pollinations o mover a `BackgroundTasks` si se vuelve un problema de UX real.
- **(2026-07-17) Rediseño de UI/UX aplicado** (paleta navy+ámbar, identidad "TVU Studio · Canal 11 UAGRM" sin logo oficial) — sustituir por assets de marca reales cuando el canal los provea.
- **(2026-07-17) Shotstack en modo `sandbox` expira los videos a las 24h** (confirmado con `curl -I`, header `x-amz-expiration`). Hace falta `SHOTSTACK_API_KEY_PRODUCTION` real y decidir si se re-aloja `output_url` en storage propio antes de la Fase 004.
- **(2026-07-17) Shotstack es composición, no IA generativa** — el spot es la imagen del personaje con efecto de cámara + subtítulo, no video con movimiento real del personaje. Decisión explícita del usuario, ver MEMORY.md. Si se quiere movimiento real, se necesita un proveedor imagen-a-video adicional.
- **(2026-07-17) Overhaul visual del frontend** (tema claro/oscuro con toggle sol/luna + loading skeletons con silueta real, ver MEMORY.md) — no era tarea del backlog fasado, se hizo por pedido directo del usuario. No se recorrió el flujo en un navegador real (sin herramienta de screenshot/automatización en este entorno) — solo se verificó `tsc -b`/`oxlint`/`npm run build` limpios y que cada módulo transforma sin error 500 vía el dev server de Vite. Recomendado que alguien con navegador confirme visualmente antes de dar por cerrado el punto "recorrer el flujo" del DoD de frontend.md.
- **(2026-07-20) `POST /api/v1/characters` no valida "imagen o descripción obligatoria"** (SOUL.md §1) — solo el frontend (`Characters.tsx`) lo exige ahora tras la mejora del formulario de creación. Llamando al endpoint directo (Swagger/script) se puede crear un personaje sin ninguno de los dos. Agregar el `if not file and not description: raise 400` en `backend/app/api/v1/characters.py` cuando se toque ese endpoint de nuevo.
- **(2026-07-17) Segunda pasada de UI/UX** (landing rediseñada, `user-select: none` en el chrome, animaciones CSS con `prefers-reduced-motion`, botones con hover/active/focus-visible, fricciones de UX en Characters/GenerateSpot/Admin — ver MEMORY.md) — tampoco era tarea del backlog fasado, pedido directo del usuario tras revisar la pasada anterior. Mismo bloqueo: sin herramienta de navegador en este entorno, verificado solo con `tsc -b`/`oxlint`/`npm run build`/`curl`. Dos pasadas de UI/UX seguidas sin confirmación visual real acumuladas — recomendado priorizar esa confirmación antes de seguir agregando cambios visuales encima.

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
- **Shotstack** (composición: imagen del personaje + efecto de cámara + subtítulo, no IA generativa) — reemplaza a "Kling Omni via Luma API" (nunca se implementó)
- Modo actual: `sandbox` (con watermark, videos expiran a las 24h)
- Documentado en MEMORY.md (entrada del 2026-07-17)
