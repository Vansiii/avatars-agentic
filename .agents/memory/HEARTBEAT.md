# HEARTBEAT — El Pulso del Proyecto

> **Foto del estado ACTUAL.** Este archivo se **sobrescribe** en cada cambio de turno.
> No es un historial: si buscas qué pasó antes, ve a `MEMORY.md`.

**Última actualización:** 2026-07-20 (cuarta pasada del día) — **Refresh token real de auth**, implementando lo que SOUL.md §9.6 ya pedía (access corto + refresh 7 días) pero nunca existía. Causa raíz de "me expulsó de la nada durante la generación": `AuthChecker` en `App.tsx` corre cada 30s y antes deslogueaba apenas vencía el access token (60 min, sin refresh) — bastaba con que la sesión completa (crear personaje + esperar un video de HeyGen, que puede tardar varios minutos) pasara de 60 min para que te pateara a mitad de una espera. Ahora: `POST /api/v1/auth/login` devuelve `access_token` (30 min) + `refresh_token` (7 días, claim `type` distingue uno de otro); `POST /api/v1/auth/refresh` cambia un refresh válido por un access nuevo; `authFetch` (`frontend/src/lib/api.ts`) renueva solo antes de cada request si el access venció, y solo fuerza logout si el REFRESH también venció o el refresh falla. `isAuthenticated()` en el store cambió de significado: antes miraba el access token, ahora mira el refresh token — es la definición correcta de "sesión viva". **Importante:** cualquier sesión ya abierta en el navegador antes de este cambio no tiene `refresh_token` guardado — la primera vez que el access token viejo venza después de este deploy, va a deslogueear una vez más; después de volver a loguearse, ya funciona sin cortes. Verificado real: login, refresh, y que un refresh token usado como access (y viceversa) se rechaza con 401. `tsc`/`oxlint`/`build` limpios.

**Actualización anterior (tercera pasada del día):** **Pollinations reemplazado por HeyGen también para la CREACIÓN de personajes** (no solo el video). Reemplazo total, decisión explícita del usuario: revierte lo que decía la entrada anterior de "no agregar selector de avatar de HeyGen". Dos caminos nuevos en `Characters.tsx`: **foto propia** (`POST /characters/create-from-photo` crea un avatar animable en HeyGen — UN solo resultado determinístico, no 3 variaciones, cuesta crédito real; el usuario confirma con `POST /characters/confirm` o descarta sin guardar nada) y **catálogo de HeyGen** (`GET /characters/heygen-catalog`, elegir directo y confirmar). `image_provider.py` (Pollinations) se borró. `Character` ganó `heygen_avatar_id`/`heygen_avatar_group_id` (migrado a mano en Supabase), usado en `POST /v3/videos type=avatar` para generar spots — con fallback a `type=image`/`reference_image_url` para los 3 personajes viejos de Pollinations (compatibilidad hacia atrás, no se rompieron). **Además, se sacó el bloqueo por límite semanal** (ya no hay 403 `CHAR_001`/`VID_001`) — el sistema es de uso interno de Canal 11 TVU (UAGRM), no un producto con plan gratuito; se mantienen los contadores solo como métrica para el admin. SOUL.md §1/§3/§4 reescritos para reflejar todo esto. Verificado real: `GET /heygen-catalog` y `POST /confirm` contra HeyGen/Supabase reales (gratis, fila de prueba borrada después); `tsc -b`/`oxlint`/`build` limpios. **NO se probó `create-from-photo` ni `POST /spots` con un personaje nuevo** (ambos cuestan crédito real) — pendiente que el usuario lo pruebe desde el frontend. Ver entrada completa en MEMORY.md.

---

## Estado Global del Proyecto

**Nuevo producto:** Sistema de Personajes para Spots Publicitarios de TV

| Fase | Estado |
|------|--------|
| Fase 001 — Base del sistema | ✅ COMPLETA |
| Fase 002 — Creación de personajes | ✅ COMPLETA |
| Fase 003 — Generación de spots | ✅ COMPLETA (ver limitación de Shotstack abajo) |
| Fase 004 — Cierre del Alpha | ⏳ Pendiente |

---

## Bloqueos actuales

1. ~~Videos de Shotstack en modo `sandbox` expiran a las 24h~~ **YA NO APLICA** (2026-07-20) — se dejó de usar Shotstack. `SHOTSTACK_*` sigue en `settings.py`/`.env` sin usar, por si se quiere volver atrás.
2. ~~Shotstack no genera movimiento de IA del personaje~~ **RESUELTO (2026-07-20)** — HeyGen anima el personaje con lip-sync real. Falta que el usuario confirme un spot real de punta a punta desde el frontend (costo real en créditos, no probado por el agente) — ver MEMORY.md.
3. ~~Pollinations.ai (free tier) rate-limita bajo ráfagas (429)~~ **YA NO APLICA** (2026-07-20) — Pollinations se dejó de usar por completo, la creación de personajes también pasa por HeyGen ahora.
4. **`HEYGEN_SPOT_VARIATIONS=1` en `.env` es temporal** (default de negocio es 3, SOUL.md §5) — el usuario lo bajó para no gastar créditos mientras prueba. Subir a 3 (o borrar la línea) apenas confirme que el flujo funciona, y reiniciar el backend (pydantic-settings solo lee `.env` al arrancar).
5. **Falta probar en vivo `POST /characters/create-from-photo` y `POST /spots` con un personaje creado por este flujo nuevo** (ambos cuestan crédito real de HeyGen) — pendiente de que el usuario lo pruebe desde el frontend.
6. **No hay endpoint para eliminar personajes** (backlog Fase 004, ítem 4.1, sigue sin implementar) — si el usuario elige mal un avatar del catálogo o confirma una foto que no le gustó después de todo, no hay forma de sacarlo de la lista desde la UI todavía.

---

## Estado Backend — Fase 003 (2026-07-17) + reescritura de Fase 002 (2026-07-20)

### Spots
- POST /api/v1/spots — genera N variaciones de video (`settings.HEYGEN_SPOT_VARIATIONS`, hoy `1`) del personaje activo + guión + tipo corto/largo; valida guión 10-500 chars, NSFW de texto fail-closed. **Ya no bloquea por límite semanal** (se sacó el 403 `VID_001`, ver SOUL.md §3).
- GET /api/v1/spots — lista spots del usuario
- GET /api/v1/spots/{id} — obtener spot
- POST /api/v1/spots/{id}/select — selecciona variación; solo aquí se incrementa `spots_used` (métrica, ya no límite)
- POST /api/v1/spots/{id}/redo — rehacer (máx 3, sin cambios — esto es sobre el VIDEO, no la creación del personaje)

### Characters (reescrito 2026-07-20, ya no usa Pollinations)
- POST /api/v1/characters/create-from-photo — sube una foto, HeyGen crea un avatar animable (`POST /v3/avatars` type=photo, base64, con polling hasta `status=completed`). Un solo resultado, NO se persiste como Character todavía.
- GET /api/v1/characters/heygen-catalog?token= — catálogo público de HeyGen paginado (`GET /v3/avatars/looks?ownership=public`).
- POST /api/v1/characters/confirm — recién acá se crea el `Character` (status `active` directo, sin "draft"); incrementa `characters_used` (métrica, ya no bloquea).
- GET /api/v1/characters, GET /api/v1/characters/{id} — sin cambios de forma (ganan `heygen_avatar_id`/`heygen_avatar_group_id` en la respuesta).
- GET /heygen-voices, PATCH /{id}/voice, GET /voice-preview — de la pasada anterior (selector de voz), sin cambios.
- **Se eliminaron:** `POST /characters` (Pollinations, 3 variaciones), `POST /characters/{id}/select`, `POST /characters/{id}/redo` — el modelo de "3 variaciones + rehacer" no aplica a HeyGen (resultado único y determinístico, cuesta crédito real). Ver SOUL.md §4.

### Servicios
- `video_provider.py` — `_submit_video` ahora arma `type: "avatar"` (con `avatar_id`) o `type: "image"` (con `reference_image_url`, fallback para personajes viejos) según qué tenga el `Character`. Nuevas `create_avatar_from_photo()` y `list_public_avatar_looks()`.
- `image_provider.py` — **borrado**, ya no se genera nada por Pollinations.
- `settings.py` — `SHOTSTACK_*` (sin uso, se dejó por si se revierte), `HEYGEN_API_KEY`, `HEYGEN_VOICE_ID` (override manual), `HEYGEN_SPOT_VARIATIONS` (default 3, hoy en `1` temporalmente en `.env`).

### DB
- `Spot.variations_data` (Text, nullable) — agregada a mano contra Supabase, mismo patrón que `characters_limit_override`.
- `Character.heygen_avatar_id`, `Character.heygen_avatar_group_id` (String, nullable) — **agregadas a mano contra Supabase 2026-07-20**, mismo patrón. `create_all()` no las hubiera creado solas en una tabla ya existente.

---

## Estado Frontend — GenerateSpot.tsx implementado (2026-07-17)

- Antes era el placeholder "Próximamente". Ahora: selector de personaje activo → guión (10-500) → tipo corto/largo → genera 3 variaciones → `<video controls>` para elegir/rehacer → lista de spots ya generados.
- Mismo patrón visual que `Characters.tsx` (loading unificado, batches de variaciones, `authFetch`).
- CSS: `.variation-card video` (16:9) y `.character-card-image video` agregadas — antes esas reglas solo cubrían `img`.
- Verificado: `tsc -b` y `npx oxlint src` limpios. No se tomaron capturas de navegador (sin herramienta de automatización en este entorno) — recomendado abrir manualmente `/spots` (ruta ya existente) para confirmar visualmente.

## Estado Frontend — Overhaul visual: tema claro/oscuro + skeletons (2026-07-17)

**Tema claro/oscuro:**
- `frontend/index.html` tiene un script inline en `<head>` que fija `data-theme` en `<html>` ANTES del primer paint (lee `localStorage["avatares-theme"]`, si no existe cae a `prefers-color-scheme`). Sin esto habría flash del tema incorrecto al recargar.
- `frontend/src/store.ts` — nuevo `useThemeStore` (zustand + `persist`, key `avatares-theme`, mismo patrón que `avatares-auth`). `toggleTheme()`/`setTheme()` escriben el atributo en `document.documentElement` directamente, sin `useEffect` de sincronización.
- `frontend/src/App.css` — `:root` sigue siendo el tema oscuro histórico (ahora documentado como tal); `:root[data-theme="light"]` define la variante clara con el mismo azul `#1a56db` / rojo `#d92626` de marca. Todos los `color: white` hardcodeados que debían seguir el tema quedaron en `var(--text)` (headings, `.card-value`, `.modal h3`, `.login-card h1`, `.sidebar-header h2`, `.landing-content h1`, `.landing-nav .brand-name`, etc.) — los que quedan literalmente blancos son texto sobre fondo sólido de color (`.btn-primary`, `.btn-danger`, `.btn-select`, `.selected-badge`), correcto en ambos temas. `--success`/`--warning` se oscurecen en el tema claro porque se usan como color de texto plano (el tono oscuro original no pasa AA sobre blanco).
- `frontend/src/components/ThemeToggle.tsx` (nuevo) — botón sol/luna compartido por `DashboardLayout` (sidebar footer, junto a "Cerrar sesión"), `Landing` (nav) y `Login` (fixed top-right, porque esas dos páginas viven fuera del `DashboardLayout`).

**Skeletons:**
- `frontend/src/components/Skeleton.tsx` (nuevo) — `CharacterCardSkeleton` (usado en `Characters.tsx` y `GenerateSpot.tsx`), `StatCardSkeleton` (usado en `Dashboard.tsx` y `Admin.tsx`), `TableRowSkeleton` (usado en `Admin.tsx`). Todos reusan las clases CSS reales (`.character-card`, `.card`, celdas de `.users-table`) + un shimmer (`.skeleton`, `@keyframes skeleton-pulse` en `App.css`).
- Reemplazan los `inline-loading` (spinner + texto) que se usaban para *fetch* de listas/métricas ya conocidas en forma: lista de personajes, lista de spots, stats del dashboard, métricas+tabla de usuarios del admin. También el spinner por-imagen de `.image-loading` (variación individual cargando en `Characters.tsx`) pasó a shimmer porque se conoce el tamaño exacto (cuadrado).
- Los spinners de generación real con IA (`.loading-overlay`: "Generando personaje...", "Generando spot...", "Generando nuevas variaciones...") NO se tocaron — tiempo variable (30-90s+), ahí un skeleton sería engañoso.
- `Dashboard.tsx` no tenía ningún estado de loading antes (mostraba 0/2 al instante); se agregó un `loading` boolean mínimo solo para poder mostrar el skeleton pedido.

**Verificación real ejecutada (pasada de tema/skeletons):**
- `npx tsc -b` limpio, `npx oxlint src` limpio (exit 0), `npm run build` genera `dist/` sin errores.
- Con backend (`:8000`, `/docs` → 200) y frontend (`:5173`) corriendo, se confirmó por `curl` que el `index.html` servido por Vite incluye el script de tema, y que los 10 archivos tocados transforman con 200 vía el dev server (un error de sintaxis da 500 con overlay ahí).
- **No se recorrió el flujo en un navegador real ni se tomaron capturas** — no hay herramienta de automatización de navegador en este entorno. Sigue pendiente en la siguiente pasada (ver abajo).

## Estado Frontend — Segunda pasada de UI/UX: landing, `user-select`, animaciones, botones (2026-07-17)

El usuario revisó la pasada anterior (tema/skeletons) y pidió una vuelta más sobre 5 ejes concretos. Todo en `frontend/src/App.css` + JSX mínimo, cero dependencias nuevas, sin tocar fetch/lógica de negocio.

**1. Landing (`frontend/src/pages/Landing.tsx`, reescrita) —** nav pasa de `position: absolute` (flotaba sobre el hero) a `position: sticky` con blur; hero pasa de una columna centrada a grid de 2 columnas (copy con eyebrow/highlight/CTA-con-ícono/lista de confianza real de SOUL.md + una tarjeta "mockup" decorativa 100% CSS, `aria-hidden`); sección nueva "Cómo funciona" con los 3 pasos reales del flujo (crear → elegir → generar); feature cards con icon-circle + hover-lift. Mismo azul `#1a56db`/rojo `#d92626` y logos reales, sin identidad nueva. Se borraron 3 clases CSS muertas que quedaron sin uso tras el rediseño (verificado con grep antes de borrar).

**2. `user-select: none` (bloque nuevo en `App.css`, justo después de `body{}`) —** por selector explícito (nada de `*{}`) sobre botones/`a`/tabs/nav/labels/badges/títulos de página y modal. Restaurado `user-select: text` explícito en `input`/`textarea`/`select`/`.users-table td`/`.character-card-info`/`.profile-info` para blindar contra herencia. Caso real detectado y corregido: `.modal-subtitle` casi quedó no-seleccionable pero ahí Admin.tsx muestra el email real del usuario — se sacó de la lista.

**3. Animaciones CSS puras** (`transition`/`@keyframes`, sin librería): `.page` (wrapper de TODAS las páginas del layout) anima fade+slide-in al montar — cubre la ruta completa sin tocar cada página; `.card`/`.character-card`/`.feature`/`.variation-card` con hover-lift consistente; botones con sombra en hover, `scale(0.97)` en `:active`, anillo en `:focus-visible`; `.modal`/`.modal-overlay` con animación de entrada (antes aparecían instantáneos); entrada escalonada (`nth-child` + `animation-delay`) en las grillas de personajes/variaciones/features/steps. Todo condicionado a `@media (prefers-reduced-motion: no-preference)`, más un bloque `reduce` al final que apaga explícitamente hover/active de tarjetas y botones.

**4. Botones —** `display: inline-flex; gap: 0.4rem` unificado para alinear ícono+texto igual en todos; `:disabled` normalizado (opacity 0.5, sin transform/sombra) en los 5 tipos de botón, no solo `.btn-primary`.

**5. Fricciones de UX corregidas —** variaciones no elegidas se atenúan (`.variation-card.dimmed`) al seleccionar una, en vez de perder su botón sin señal visual (`Characters.tsx`/`GenerateSpot.tsx`, una clase condicional, sin tocar el flujo). `Admin.tsx`: botones de solo-texto ("Límites", "Eliminar", "Editar", "Crear Usuario/Categoría", "Guardar"/"Cancelar"/"Cerrar") ganaron ícono + `title`, para consistencia con Characters/GenerateSpot que ya usaban ícono+texto en todo.

**Verificación real ejecutada:**
- `npx tsc -b` limpio, `npx oxlint src` limpio (exit 0), `npm run build` compila sin error.
- Con backend (`:8000`) y frontend (`:5173`) corriendo, se confirmó por `curl` que los 5 archivos tocados transforman con 200 vía el dev server de Vite. Se verificó balance de llaves de `App.css` con un script Node (profundidad final 0) tras todas las ediciones.
- **Sigue sin haber herramienta de automatización de navegador en este entorno** (mismo bloqueo que la pasada anterior) — no se tomaron capturas ni se recorrió el flujo clickeando en un navegador real. Pendiente que alguien con navegador confirme visualmente: landing (hero de 2 columnas, mockup, sección "Cómo funciona"), que `user-select` no rompió selección en inputs/tablas/nombre de personaje, tema claro sobre las clases nuevas, y las animaciones (fade-in de página, hover de tarjetas, entrada del modal, stagger de listas).

---

## Verificación real ejecutada (no solo "compila")

Con confirmación explícita del usuario: se corrió la migración contra el Supabase real y se probó `POST /api/v1/spots` contra el sandbox real de Shotstack (no mockeado) — 3 archivos `.mp4` reales devueltos en ~14s, confirmados con `curl -I` (`Content-Type: video/mp4`). `select` decrementó el límite y persistió `output_url`. Casos negativos correctos: guión corto (400), tipo inválido (400), guión con palabra bloqueada (422 `GEN_004`, logueado), personaje no activo (400).

---

## Credenciales

- **Admin:** admin@avatares.com / admin123
- **Backend:** http://localhost:8000 (Swagger: /docs)
- **Frontend:** http://localhost:5173

---

## Pendiente para Fase 004

- [ ] Gestión de personajes: editar, eliminar, reasignar categoría — **más urgente desde 2026-07-20**: si el usuario confirma un personaje por error (foto o catálogo), hoy no hay forma de sacarlo de la lista
- [ ] Dashboard con métricas de uso (admin)
- [ ] Tests E2E del flujo completo
- [ ] Eliminación de imágenes/videos a las 24h + strip de EXIF (SOUL.md §7)
- [ ] ~~Decidir si se persiste `output_url` de Shotstack...~~ ya no aplica (se dejó Shotstack) — **nuevo:** verificar si `video_url` de HeyGen expira y si hace falta re-alojarlo en storage propio (no verificado todavía)
- [ ] ~~Conseguir `SHOTSTACK_API_KEY_PRODUCTION`~~ ya no aplica, se dejó Shotstack por HeyGen (2026-07-20)
- [ ] Subir `HEYGEN_SPOT_VARIATIONS` de `1` a `3` en `.env` cuando el usuario confirme que el flujo de spots con HeyGen funciona de punta a punta

---

## Para el siguiente agente

1. Leer SOUL.md antes de empezar
2. Para levantar: `backend/.venv` ya existe — `uvicorn app.main:app --reload` desde `backend/`
3. Frontend: `npm run dev` desde `frontend/`
4. Usar `authFetch` de `frontend/src/lib/api.ts` para TODAS las llamadas autenticadas
5. Límites y `week_start` viven en `app/services/limits.py` — no los reintroduzcas duplicados
6. Si agregas columnas a modelos existentes: `create_all()` no las migra en tablas ya creadas en Supabase — hay que correr el `ALTER TABLE` a mano (con confirmación del usuario)
7. Shotstack (`video_provider.py`) es composición, no IA generativa — no asumas que puede generar movimiento de un personaje sin un proveedor imagen-a-video previo
8. Tema claro/oscuro: `useThemeStore` en `frontend/src/store.ts`, toggle en `frontend/src/components/ThemeToggle.tsx`. Pendiente que alguien confirme visualmente en navegador (no hubo herramienta de screenshot en la sesión que lo implementó, 2026-07-17).
9. Loading: para listas/métricas con forma conocida, usá los skeletons de `frontend/src/components/Skeleton.tsx`, no un spinner genérico — ese patrón queda solo para generación de IA de tiempo variable (`.loading-overlay`)
10. `user-select: none` del chrome vive en un bloque explícito en `App.css` justo después de `body{}` — si agregás un botón/badge/título de UI nuevo, sumalo ahí por selector (NO uses `* { user-select: none }`); si agregás algo que muestre datos reales del usuario (email, nombre, descripción), asegurate de que no quede alcanzado por herencia — hay un segundo bloque justo debajo que restaura `user-select: text` en los contenedores de datos conocidos.
11. Animaciones nuevas van condicionadas a `@media (prefers-reduced-motion: no-preference)` (o apagadas explícitamente en el bloque `reduce` al final de `App.css`) — no agregues un `@keyframes` que se dispare siempre sin ese guard.
12. **(RESUELTO 2026-07-18)** El usuario confirmó el frontend en navegador real (`localhost:5173`, "Está bien el frontend") — se cierra el punto "recorrer el flujo en el navegador" del DoD de frontend.md que arrastraban las dos pasadas de 2026-07-17. Sigue sin haber herramienta de automatización de navegador en el entorno del agente (las verificaciones automáticas siguen siendo `tsc`/`oxlint`/`build`/`curl`), pero la confirmación visual humana ya está hecha sobre el estado acumulado.
13. Siguiente fase: **Fase 004 — Cierre del Alpha**
14. **(2026-07-20) Proveedor de video ahora es HeyGen**, no Shotstack — `video_provider.py` reescrito, ver MEMORY.md. `HEYGEN_SPOT_VARIATIONS=1` en `.env` es temporal (subir a 3 cuando el usuario confirme el flujo, y reiniciar el backend).
15. **(2026-07-20, reversa lo que decía el punto 14) La creación de personajes TAMBIÉN pasa por HeyGen ahora** (foto propia o catálogo) — Pollinations/`image_provider.py` se eliminó por completo. Si tocás `Characters.tsx` o `characters.py`, el flujo es: `create-from-photo`/`heygen-catalog` → `confirm`. No hay más "3 variaciones + rehacer" para personajes (sí sigue para spots/video). Ver SOUL.md §4 y MEMORY.md 2026-07-20 para el porqué del giro.
16. **(2026-07-20) Los límites semanales (SOUL.md §3) ya NO bloquean** — no busques ni reintroduzcas los códigos `CHAR_001`/`VID_001`, se sacaron a propósito (uso interno de Canal 11 TVU, no plan gratuito). Los contadores siguen sumando solo para las métricas del admin.
17. **Pendiente de probar con costo real:** `create-from-photo` y `POST /spots` sobre un personaje creado por el flujo nuevo — nadie los corrió todavía (consumen crédito real de HeyGen), el usuario dijo que los prueba él mismo desde el frontend.
18. **(2026-07-20) Refresh token real de auth** — `access_token` (30 min) + `refresh_token` (7 días), `POST /api/v1/auth/refresh`. `authFetch` renueva solo, `isAuthenticated()` del store ahora mira el refresh token (antes miraba el access, por eso deslogueaba a media generación de video). Si tocás algo de auth: `decode_access_token` ya no existe, se llama `decode_token` (genérico, distingue por claim `type`). Ver MEMORY.md.
