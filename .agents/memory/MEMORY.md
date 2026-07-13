# MEMORY — Historial Inmutable

> **Log append-only.** Escribe al FINAL, nunca edites ni borres entradas anteriores.
> Aunque una decisión pasada fuera un error, se queda: se corrige con una entrada nueva que la revierte.
>
> Formato de cada entrada:
>
> ```
> ## [FECHA] — Agente <Rol> — <Título corto>
> **Hice:** qué se completó.
> **Archivos:** rutas tocadas.
> **Decisión:** qué se eligió y por qué (si aplica).
> **Para el siguiente agente:** lo que necesita saber para no romper esto.
> ```

---

## [2026-07-13] — Agente Orquestador — Instalación del harness multi-agente

**Hice:** monté la estructura `.agents/` del harness. Antes existían `memory/` y `steering/` anidados dentro de `.agents/skills/` y con los archivos **vacíos**, así que la memoria compartida no existía en la práctica: cada agente arrancaba a ciegas.

**Archivos:** creados `.agents/AGENTS.md`, `.agents/memory/{SOUL,HEARTBEAT,MEMORY}.md`, `.agents/steering/backlog.md`; escritos `.agents/skills/{frontend,backend,ai}.md`, que también estaban vacíos.

**Decisión:** `memory/` y `steering/` se colocan como hermanas de `skills/`, colgando de `.agents/`, tal como manda el informe de revisión. Una carpeta de memoria dentro de `skills/` se leería como una skill más, no como estado compartido.

**Para el siguiente agente:** lee `SOUL.md` antes de tocar nada. Las tres reglas que más fácil se violan sin darse cuenta: **watermark en todo avatar del Alpha**, **filtro NSFW obligatorio en entrada y salida**, y **cero Celery/Redis**.

---

## [2026-07-13] — Agente Orquestador — Auditoría del estado real del código

**Hice:** revisé `backend/` y `frontend/` para anclar el HEARTBEAT a la realidad y no a lo que dicen las specs.

**Hallazgos que todo agente debe conocer:**

1. **La generación de avatares es un mock.** `backend/app/api/v1/generations.py::generate_avatar_background` no llama a ninguna IA: hace `asyncio.sleep` y devuelve URLs fijas de Unsplash. El flujo completo (202 → WebSocket → historial) funciona de punta a punta, pero las imágenes son falsas.
2. **No hay filtro NSFW en ninguna parte.** Viola `SOUL.md §4`. Es el bloqueo más grave para cerrar el Alpha.
3. **El watermark es solo una bandera.** `is_watermarked=True` se guarda en la base de datos, pero no se estampa ningún píxel sobre la imagen.
4. **El stack real del frontend difiere de las specs:** es React **19** + TypeScript + zustand + react-query con CSS global, no React 18 con CSS Modules. Manda el código: `frontend/package.json`.
5. **El backend ya cumple KISS:** no hay Celery ni Redis; se usa `BackgroundTasks`. La sobreingeniería que denuncia el informe está en los documentos (`spec/sdd.md`, `spec/roadmap.md`), no en el código. **No sigas el roadmap al pie de la letra**: su hito 2.1 pide Celery + Redis, y `SOUL.md` lo prohíbe.

**Decisión:** el contrato de API vigente entre Frontend y Backend queda fijado así (fuente: `app/schemas/generation.py` y las rutas reales):

| Método | Ruta | Cuerpo / Respuesta |
|---|---|---|
| `POST` | `/api/v1/generations` | `multipart/form-data`: `style_id` (UUID), `prompt?`, `variations` (3–6, def. 3), `notes?`, `file?`. → **202** `{status, data:{request_id, estimated_seconds, websocket_channel}}` |
| `WS` | `/api/v1/ws/generations/{request_id}` | eventos `generation_progress` · `generation_completed` · `generation_failed` |
| `GET` | `/api/v1/users/me/history?page&limit` | lista de solicitudes con sus `avatars[]` |

Errores ya implementados: `GEN_001` (formato inválido), `GEN_002` (>10 MB), `GEN_003` (sin créditos). Falta `GEN_004` (NSFW).

**Para el siguiente agente:** si cambias cualquier fila de esa tabla, **añade una entrada nueva aquí**. El Frontend construye contra ese contrato y no tiene otra forma de enterarse.

---

## [2026-07-13] — Agente IA — Generación real con Pollinations.ai

**Hice:** reemplacé el mock de Unsplash por generación de imágenes real. Se evaluaron tres proveedores antes de decidir:

- **Replicate:** pay-as-you-go, requiere tarjeta, ~$0.01–0.05/imagen con SDXL.
- **Gemini API:** $0.039/imagen con Gemini 2.5 Flash Image; trae filtros de seguridad nativos (`blockReason SAFETY` en entrada, `IMAGE_SAFETY` en salida) que habrían cubierto los dos checkpoints de `SOUL.md §4` de una vez; su watermark **SynthID es invisible**, no sirve como el watermark visible que pide el producto.
- **Pollinations.ai (elegido):** gratuito, sin tarjeta, funciona incluso en modo anónimo (verificado sin API key). Solo filtra NSFW en la **entrada** (`safe=true`); no hay filtro documentado de salida — brecha que queda abierta a propósito, no por descuido.

**Archivos:**
- Nuevos: `backend/app/media_paths.py`, `backend/app/services/{image_provider,watermark,security_log}.py`
- Modificados: `backend/app/api/v1/generations.py` (pipeline real dentro de `generate_avatar_background`, mismos eventos de WebSocket, sin cambios de contrato), `backend/app/main.py` (mount de `/media` con `StaticFiles`), `backend/app/config/settings.py` (+`POLLINATIONS_API_KEY`), `backend/.env` (+línea vacía para la key), `backend/requirements.txt` (+`pillow`), `backend/.gitignore` (+`app/media/`), `frontend/vite.config.ts` (+proxy de `/media`)

**Decisión:** los avatares se guardan en disco local (`app/media/avatars/`) servidos por `StaticFiles`, no en S3/CDN — consistente con "sin infraestructura nueva en el Alpha". El `cdn_url` guardado es una ruta relativa (`/media/avatars/...`); esto **solo funciona si frontend y backend comparten origen a través de un proxy** (Vite lo hace en dev). Si se despliega a producción sin ese patrón, hay que revisarlo.

**Decisión (del usuario, explícita):** foto→avatar personalizado queda **diferido**. El modelo `kontext` de Pollinations exige URL pública de la imagen de entrada, y el proyecto no tiene storage público. Por ahora, subir solo una foto sin prompt genera un avatar del **estilo elegido**, no personalizado — está comentado en el código, no es un bug silencioso.

**Verificación real (no solo "compila"):** llamé a `generate_image()` en vivo, descargué una imagen JPEG real de Pollinations, la pasé por `apply_watermark()` y **miré el PNG resultante** — el texto "ProyectoIA · Alpha" es legible en la esquina inferior derecha con buen contraste sobre cualquier fondo.

**Hallazgo colateral importante:** al revisar `Generate.tsx` para confirmar el contrato de WebSocket, encontré que **ya estaba cableado contra el backend** desde antes — el HEARTBEAT anterior asumía que faltaba hacerlo (tarea C-01) y eso era incorrecto; quien escribió ese HEARTBEAT no verificó el frontend a fondo. También encontré un bug de producto no relacionado con esta tarea: `socket.onerror` dispara un `runSimulation()` que muestra fotos de stock como si fueran el resultado real, sin avisar al usuario. Era inofensivo con el mock; ahora que la generación es real, es engañoso. Lo dejé documentado como `C-08` en el backlog, sin arreglarlo — no es responsabilidad del Agente IA tocar código de frontend.

**Para el siguiente agente:** el filtro NSFW de **salida** (`B-04`) sigue sin existir. No asumas que `safe=true` de Pollinations cubre la imagen ya generada — solo protege la generación misma, no hay confirmación de que revise el resultado. Si vas a cerrar esa brecha, necesitas un servicio de moderación de imágenes aparte.
