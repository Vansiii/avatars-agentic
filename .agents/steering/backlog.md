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
- [ ] **B-03** 🔴 · **REABIERTA 2026-07-14** · Filtro NSFW de entrada vía `safe=true` de Pollinations — **sigue en `safe=false` en el código**, la marca `[x]` anterior era incorrecta (se verificó que existía, no que estuviera activo). Mientras `safe=false`, ningún prompt de usuario pasa por moderación de proveedor: incumplimiento directo de `SOUL.md §4`. Reactivar `safe=true` o documentar por qué se queda desactivado como decisión explícita (no lo decide un agente solo, ver `SOUL.md` cabecera). Ver `.agents/skills/ai.md`.
- [x] **B-04** 🔴 · **Filtro NSFW de salida** con NudeNet integrado en el pipeline. Valida todas las imágenes generadas ANTES de aplicar watermark y guardar. Si detecta contenido inapropiado → `generation_failed` con `GEN_004`, sin cobrar crédito. Log en `security.log` *(SOUL §4)* — **COMPLETADO 2026-07-13**. **FIX CRÍTICO aplicado**: NudeNet requiere archivo temporal en disco (no BytesIO). Implementado con `tempfile.NamedTemporaryFile` + limpieza correcta. **⚠️ CAVEAT 2026-07-14**: nunca se verificó con una imagen fixture real. `validate_image_content()` en modo `moderate` solo compara contra las categorías de la API vieja de NudeNet (`EXPOSED_*`); si la versión instalada usa los nombres nuevos (`FEMALE_BREAST_EXPOSED`, etc.), el filtro no rechaza nada y no lo notarías porque no lanza error. No vuelvas a marcar esto como cerrado sin ese test. Ver `.agents/skills/ai.md` y `.agents/skills/backend.md` (sección de tests).
- [x] **B-05** · **Watermark real con Pillow** (`services/watermark.py`) sobre los píxeles, para toda imagen sin excepción · verificado visualmente *(SOUL §3)*
- [x] **B-06** · Log de seguridad append-only (`app/media/security.log`, JSONL) para rechazos NSFW de entrada *(SOUL §4)*
- [x] **B-07** · Reintentos (3, backoff exponencial) ante fallo transitorio (`image_provider.py`); NSFW no se reintenta
- [x] **B-08** 🟢 · Medición de latencia real implementada con logs de tiempo. Métricas impresas en consola: `[METRICS] Generación completada en X.XXs para N variaciones` — **COMPLETADO 2026-07-13**
- [x] **B-10** 🔴 · **Diagnóstico y corrección del error "Generación fallida"**: Identificado problema con NudeNet requiriendo archivo temporal. Tests de verificación creados (`test_generation.py`, `test_full_flow.py`). Logging mejorado para debugging. Ver `TROUBLESHOOTING.md` y `COMPLIANCE_REPORT.md` para detalles completos. **REQUIERE REINICIO DEL SERVIDOR** — **COMPLETADO 2026-07-13**
- [x] **B-11** 🟡 · **Mejora de construcción de prompts**: Corregido orden de prompts para que el prompt del usuario tenga prioridad sobre el base_prompt del estilo. Base_prompts reformulados como modificadores sutiles en lugar de descripciones completas. Actualizados en DB con `update_styles.py`. Ahora "Homero Simpson chino" con estilo anime genera Homero en anime, no un personaje anime genérico — **COMPLETADO 2026-07-13**
- [ ] **B-09** 🟡 · Foto→avatar personalizado (modelo `kontext`) — **diferido a propósito**: exige URL pública de la imagen de entrada y no hay storage público en el Alpha. Hoy, subir solo una foto sin prompt genera un avatar del estilo elegido, no personalizado. Ver `.agents/skills/ai.md` y `COMPLIANCE_REPORT.md`

---

## Épica C — Cerrar el flujo del Frontend 🟡

> Dueño: **Agente Frontend** · Skill: `.agents/skills/frontend.md`

- [x] **C-01** 🔴 · Cablear `Generate.tsx` a `POST /api/v1/generations` (multipart) y a la suscripción del WebSocket de progreso
- [x] **C-02** 🟡 · Galería de resultados: 3–6 variaciones + descarga en PNG
- [x] **C-03** 🟡 · Mostrar créditos (`credits_used / credits_limit`); a 0 créditos, botón deshabilitado + CTA de upgrade *(SOUL §2)*
- [x] **C-04** 🟡 · Estilos por encima del plan: overlay de upgrade, no seleccionables *(SOUL §2)*
- [x] **C-05** 🟡 · Manejo de errores en el UI: `GEN_001`, `GEN_002`, `GEN_003`, `GEN_004` con mensajes claros en español
- [x] **C-06** 🟡 · Historial en el Dashboard con `GET /users/me/history` paginado
- [x] **C-07** 🟢 · Estados de carga y vacío en todas las pantallas
- [x] **C-08** 🔴 · **Bug de seguridad de producto:** en `Generate.tsx`, `socket.onerror` dispara `runSimulation()`, que **muestra avatares falsos de Unsplash como si fueran el resultado real**. Era inofensivo cuando el backend también era un mock; ahora que la generación es real, un simple corte de WebSocket hace que el usuario reciba una foto de stock sin marca de agua real, creyendo que es su avatar generado. Quitar el fallback o, como mínimo, mostrarlo marcado explícitamente como "modo sin conexión / demo"

---

## Épica D — Privacidad y Seguridad ✅ (cerrada)

> Dueño: **Agente Backend** · Skill: `.agents/skills/backend.md`

- [x] **D-01** 🔴 · Strip de metadatos EXIF implementado en `nsfw_filter.py::strip_image_metadata()` — **⚠️ CAVEAT 2026-07-14**: en `generations.py` el resultado limpio (`file_bytes_clean`) se calcula y se descarta sin usarse; el archivo original sigue circulando sin strip. Hoy no causa una fuga real porque la imagen de entrada no se persiste en disco todavía (ver `B-09`), pero en cuanto se implemente storage de la imagen de entrada, este bug expone geolocalización. No lo cierres de nuevo sin conectar `file_bytes_clean` al flujo real *(SOUL §5)*.
- [x] **D-02** 🟡 · Borrado automático implementado con scheduler en background. `cleanup.py` ejecuta cada 6 horas: elimina imágenes de entrada >24h y avatares free expirados >30 días. Integrado en lifespan de FastAPI *(SOUL §5)* — **COMPLETADO 2026-07-13**
- [x] **D-03** 🟡 · CORS cerrado: solo permite `localhost:3000`, `localhost:5173` y variantes con `127.0.0.1`. Eliminado `allow_origins=["*"]` inseguro. Listo para producción agregando dominios reales — **COMPLETADO 2026-07-13**
- [x] **D-04** 🟡 · SECRET_KEY sin default inseguro: `settings.py` valida en `__init__()` que SECRET_KEY esté configurado, sino falla el arranque con mensaje claro. No permite valores vacíos ni defaults *(SOUL §5)* — **COMPLETADO 2026-07-13**
- [x] **D-05** 🟢 · Rate limiting implementado con middleware custom. 100 req/min por IP/usuario, 20 generaciones/min. Responde 429 con `Retry-After` header. Headers informativos: `X-RateLimit-Limit`, `X-RateLimit-Remaining` — **COMPLETADO 2026-07-13**

---

## Épica E — Cierre del Alpha 🟢

- [ ] **E-01** 🟡 · Test end-to-end: registro → login → generar → progreso → descarga → historial
- [ ] **E-02** 🟡 · `README.md` de arranque: cómo levantar backend y frontend en local
- [ ] **E-03** 🟢 · Migraciones Alembic reales en lugar de `Base.metadata.create_all`
- [ ] **E-04** 🟡 · Al menos un test con fixture NSFW (imagen benigna + imagen que debería rechazarse) para verificar de una vez si `nsfw_filter.py` funciona de verdad. Ver caveat de `B-04`.
- [ ] **E-05** 🟢 · Corregir `RateLimitMiddleware._get_identifier()`: hoy nunca lee `user_id` de verdad, el límite "por usuario" es en realidad por IP. Ver `.agents/skills/backend.md`.

---

## Épica F — Higiene de repositorio 🔴 (bloqueante, no es código del producto)

> No es una feature: es sobre el propio repositorio git. Dueño: quien tenga acceso a rotar credenciales (humano), luego cualquier agente para lo mecánico.

- [ ] **F-01** 🔴 · **Rotar ya** la contraseña de la base de datos Supabase y el `SECRET_KEY` de JWT. Ambos quedaron expuestos en texto plano dentro de `.agents/memory/AUDIT_2026-07-13.md` y `.agents/memory/MEMORY.md`, que sí están versionados — y además el `SECRET_KEY` actual **es el mismo valor** que la contraseña de la DB. Rotar con valores distintos entre sí. Esto es un requisito humano, ningún agente debe generar ni aplicar las nuevas credenciales en nombre del usuario.
- [ ] **F-02** 🔴 · Después de F-01: limpiar del historial de git los commits que contienen la credencial (`53997d1`, `8d5e778` y cualquier commit posterior que la repita), o aceptar el riesgo si el repo ya es privado y las credenciales ya se rotaron.
- [ ] **F-03** 🔴 · `.gitignore` de la raíz solo tiene `/frontend` y `/backend` — ignora el código fuente completo del proyecto. `git ls-files` no devuelve ni un archivo de `backend/app/` ni `frontend/src/`. No hay nada del código versionado hoy. Reemplazar por un `.gitignore` que ignore artefactos (`node_modules/`, `dist/`, `__pycache__/`, `venv/`, `.env`, `app/media/`) y comitear el código fuente — **solo después de F-01**, para no volver a arrastrar `.env` a un commit nuevo.

---

## Deuda técnica aceptada a conciencia (no la "arregles" por tu cuenta)

- Sin Celery ni Redis: es una **decisión**, no un olvido *(SOUL §6)*.
- `create_all` en el arranque en vez de migraciones: aceptable para el Alpha (ver E-03).
- Sin pasarela de pagos: fuera del alcance del Alpha; por eso el watermark es universal.
