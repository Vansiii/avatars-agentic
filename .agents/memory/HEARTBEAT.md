# HEARTBEAT — El Pulso del Proyecto

> **Foto del estado ACTUAL.** Este archivo se **sobrescribe** en cada cambio de turno.
> No es un historial: si buscas qué pasó antes, ve a `MEMORY.md`.

**Última actualización:** 2026-07-17 — Segunda pasada de UI/UX del frontend: landing rediseñada (hero de 2 columnas + mockup CSS + sección "Cómo funciona"), `user-select: none` aplicado al "chrome" de la UI (botones/nav/tabs/badges) sin tocar contenido real (inputs, emails, tablas, nombre/descripción de personaje), animaciones CSS puras (fade-in de página, hover-lift de tarjetas, press de botones, transición de modal, entrada escalonada de listas) todas guardadas con `prefers-reduced-motion`, y botones con estados hover/active/focus-visible/disabled consistentes. Tarea puramente visual, sin tocar backend ni contrato de API. Ver sección "Estado Frontend" abajo.

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

1. **Videos de Shotstack en modo `sandbox` expiran a las 24h** (`x-amz-expiration: Delete After 24 Hours` en el S3 de salida, confirmado con `curl -I` real). Un spot seleccionado hoy deja de tener `output_url` funcional mañana. Para producción real hace falta `SHOTSTACK_API_KEY_PRODUCTION` (hoy `xxxxx` en `.env`) y, probablemente, descargar/re-alojar el video en storage propio antes de que expire. No implementado — anotado para Fase 004.
2. **Shotstack no genera movimiento de IA del personaje** — es un motor de composición. El usuario pidió explícitamente un avatar animado real ("que interactúe a lo largo del video") y se le presentaron opciones investigadas en vivo (HeyGen recomendado, Hedra alternativa) — respondió **"Ninguno por ahora"**, así que se mejoró el montage dentro de Shotstack (varios planos/encuadres de la misma foto + fundidos + subtítulos segmentados) en vez de sumar proveedor. Si el usuario cambia de opinión, retomar la comparación ya hecha en `MEMORY.md` (re-verificar contra docs oficiales, el campo cambia rápido) — hace falta sumar un proveedor imagen-a-video/foto-a-avatar ANTES de Shotstack, no en su lugar.
3. **Pollinations.ai (free tier) rate-limita bajo ráfagas (429).** Sigue igual que antes — no relacionado a Fase 003.

---

## Estado Backend — Fase 003 agregada (2026-07-17)

### Spots (nuevo)
- POST /api/v1/spots — genera 3 variaciones de video (personaje activo + guión + tipo corto/largo); valida guión 10-500 chars, NSFW de texto fail-closed, límite semanal (`VID_001`)
- GET /api/v1/spots — lista spots del usuario
- GET /api/v1/spots/{id} — obtener spot
- POST /api/v1/spots/{id}/select — selecciona variación; solo aquí se decrementa `spots_used`
- POST /api/v1/spots/{id}/redo — rehacer (máx 3, no decrementa límite)

### Servicios
- `video_provider.py` — **reescrito**: llama a Shotstack real (`build_spot_edit`, `_submit_render`, `_poll_render`, `generate_spot_variations`). Antes era un placeholder que devolvía URLs falsas. Cada spot lleva narración con el `text-to-speech` nativo de Shotstack (voz Lupe, es-US, modo newscaster) — no un proveedor externo (Pollinations TTS quedó deprecado, verificado en vivo).
- `settings.py` — nuevas variables `SHOTSTACK_API_KEY_SANDBOX`, `SHOTSTACK_API_KEY_PRODUCTION`, `SHOTSTACK_ENV`.

### DB
- `Spot.variations_data` (Text, nullable) — **agregada a mano contra Supabase** (`ALTER TABLE spots ADD COLUMN IF NOT EXISTS variations_data TEXT`), mismo patrón que `characters_limit_override`. `create_all()` no la hubiera creado sola en una tabla ya existente.

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

- [ ] Gestión de personajes: editar, eliminar, reasignar categoría
- [ ] Dashboard con métricas de uso (admin)
- [ ] Tests E2E del flujo completo
- [ ] Eliminación de imágenes/videos a las 24h + strip de EXIF (SOUL.md §7)
- [ ] Decidir si se persiste `output_url` de Shotstack en storage propio antes de que expire (24h en sandbox)
- [ ] Conseguir `SHOTSTACK_API_KEY_PRODUCTION` si se quiere salir de sandbox (sin watermark, sin expiración de 24h)

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
12. Sigue sin haber herramienta de automatización de navegador en este entorno (ni en la pasada de tema/skeletons ni en la de landing/animaciones/`user-select`, ambas 2026-07-17) — dos rounds seguidos de UI/UX verificados solo con `tsc`/`oxlint`/`build`/`curl`. Pendiente que alguien con navegador real confirme visualmente todo lo acumulado antes de dar por cerrado el punto "recorrer el flujo en el navegador" del DoD de frontend.md.
13. Siguiente fase: **Fase 004 — Cierre del Alpha**
