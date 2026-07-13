# Backlog del Alpha

> El Orquestador elige de aquí la siguiente tarea desbloqueada y marca `[ ]` → `[x]` al terminarla.
> **Regla:** una tarea solo se marca cuando se ha **verificado ejecutándola**, no cuando el código "se ve bien".
>
> **Este backlog manda sobre `spec/roadmap.md`.** El roadmap pide Celery + Redis (hito 2.1) y `SOUL.md §6` lo prohíbe. Ante conflicto, gana SOUL.

**Leyenda:** 🔴 bloqueante para el Alpha · 🟡 importante · 🟢 mejora

---

## Épica A — Fundaciones ✅ (cerrada)

- [x] **A-01** · Backend: monolito FastAPI con CORS y lifespan · `backend/app/main.py`
- [x] **A-02** · Backend: modelos `User`, `Style`, `GenerationRequest`, `GeneratedAvatar`
- [x] **A-03** · Backend: auth con JWT (registro, login, `get_current_user`)
- [x] **A-04** · Backend: catálogo de estilos + seed en el arranque
- [x] **A-05** · Backend: `POST /generations` (202) + WebSocket de progreso + `GET /users/me/history`
- [x] **A-06** · Backend: reglas de créditos y de tier de estilo (403 `GEN_003`)
- [x] **A-07** · Frontend: React 19 + Vite + rutas + páginas base (Landing/Login/Register/Dashboard/Generate/Profile)
- [x] **A-08** · Harness: estructura `.agents/` con memoria compartida

---

## Épica B — IA Real 🔴 (el Alpha NO existe sin esto)

> Dueño: **Agente IA** · Skill: `.agents/skills/ai.md`
> **Proveedor: Pollinations.ai** (gratis), no Replicate ni Gemini — decisión del 2026-07-13.

- [x] **B-01** · `POLLINATIONS_API_KEY` en `app/config/settings.py` + `pillow` en `requirements.txt` (Pollinations no requiere cliente propio, se usa `httpx` que ya existía)
- [x] **B-02** · Mock de Unsplash sustituido por la llamada real a `image.pollinations.ai` en `generate_avatar_background`, eventos del WebSocket conservados sin cambios · verificado en vivo (imagen real descargada y guardada)
- [x] **B-03** · **Filtro NSFW de entrada** vía `safe=true` de Pollinations, fail-closed (cualquier 4xx = rechazo) → llega como evento `generation_failed` con `error_code: GEN_004` por WebSocket, **sin cobrar crédito** *(SOUL §4)* — nota: no es un 422 HTTP directo porque el POST ya respondió 202 antes de saberse el resultado
- [ ] **B-04** 🔴 · **Filtro NSFW de salida** sobre la imagen ya generada. Pollinations **no** documenta un filtro de salida — sigue siendo una brecha real contra `SOUL.md §4`. Requiere integrar un servicio de moderación de imágenes aparte
- [x] **B-05** · **Watermark real con Pillow** (`services/watermark.py`) sobre los píxeles, para toda imagen sin excepción · verificado visualmente *(SOUL §3)*
- [x] **B-06** · Log de seguridad append-only (`app/media/security.log`, JSONL) para rechazos NSFW de entrada *(SOUL §4)*
- [x] **B-07** · Reintentos (3, backoff exponencial) ante fallo transitorio (`image_provider.py`); NSFW no se reintenta
- [ ] **B-08** 🟢 · Anotar coste/latencia reales — coste es $0 al ser Pollinations gratuito; falta medir la **latencia real** con varias generaciones seguidas y confirmar que se mantiene bajo 60s con `variations=6`
- [ ] **B-09** 🟡 · Foto→avatar personalizado (modelo `kontext`) — **diferido a propósito**: exige URL pública de la imagen de entrada y no hay storage público en el Alpha. Hoy, subir solo una foto sin prompt genera un avatar del estilo elegido, no personalizado. Ver `.agents/skills/ai.md`

---

## Épica C — Cerrar el flujo del Frontend 🟡

> Dueño: **Agente Frontend** · Skill: `.agents/skills/frontend.md`

- [ ] **C-01** 🔴 · Cablear `Generate.tsx` a `POST /api/v1/generations` (multipart) y a la suscripción del WebSocket de progreso
- [ ] **C-02** 🟡 · Galería de resultados: 3–6 variaciones + descarga en PNG
- [ ] **C-03** 🟡 · Mostrar créditos (`credits_used / credits_limit`); a 0 créditos, botón deshabilitado + CTA de upgrade *(SOUL §2)*
- [ ] **C-04** 🟡 · Estilos por encima del plan: overlay de upgrade, no seleccionables *(SOUL §2)*
- [ ] **C-05** 🟡 · Manejo de errores en el UI: `GEN_001`, `GEN_002`, `GEN_003`, `GEN_004` con mensajes claros en español
- [ ] **C-06** 🟡 · Historial en el Dashboard con `GET /users/me/history` paginado
- [ ] **C-07** 🟢 · Estados de carga y vacío en todas las pantallas
- [ ] **C-08** 🔴 · **Bug de seguridad de producto:** en `Generate.tsx`, `socket.onerror` dispara `runSimulation()`, que **muestra avatares falsos de Unsplash como si fueran el resultado real**. Era inofensivo cuando el backend también era un mock; ahora que la generación es real, un simple corte de WebSocket hace que el usuario reciba una foto de stock sin marca de agua real, creyendo que es su avatar generado. Quitar el fallback o, como mínimo, mostrarlo marcado explícitamente como "modo sin conexión / demo"

---

## Épica D — Privacidad y Seguridad 🟡

> Dueño: **Agente Backend** · Skill: `.agents/skills/backend.md`

- [ ] **D-01** 🔴 · Strip de metadatos EXIF en toda imagen subida *(SOUL §5)*
- [ ] **D-02** 🟡 · Borrado automático de las imágenes de entrada a las 24 h *(SOUL §5)*
- [ ] **D-03** 🟡 · Cerrar CORS: `allow_origins` con la lista real de dominios, no `["*"]`
- [ ] **D-04** 🟡 · Quitar el valor por defecto inseguro de `SECRET_KEY` en `settings.py`; que falle el arranque si no está definida
- [ ] **D-05** 🟢 · Rate limiting por usuario e IP (429 con `Retry-After`)

---

## Épica E — Cierre del Alpha 🟢

- [ ] **E-01** 🟡 · Test end-to-end: registro → login → generar → progreso → descarga → historial
- [ ] **E-02** 🟡 · `README.md` de arranque: cómo levantar backend y frontend en local
- [ ] **E-03** 🟢 · Migraciones Alembic reales en lugar de `Base.metadata.create_all`

---

## Deuda técnica aceptada a conciencia (no la "arregles" por tu cuenta)

- Sin Celery ni Redis: es una **decisión**, no un olvido *(SOUL §6)*.
- `create_all` en el arranque en vez de migraciones: aceptable para el Alpha (ver E-03).
- Sin pasarela de pagos: fuera del alcance del Alpha; por eso el watermark es universal.
