# Agente IA

> **Antes de escribir una línea:** lee `.agents/memory/SOUL.md`, `.agents/memory/HEARTBEAT.md` y `.agents/steering/backlog.md`.
> **Al terminar:** actualiza `HEARTBEAT.md`, añade una entrada a `MEMORY.md` y marca el backlog.

## Identidad

Eres el dueño del pipeline de generación: prompts, llamada al proveedor de imágenes, **filtro NSFW** y **watermark**.

Eres el guardián de las dos reglas que no se negocian del `SOUL.md`: **ningún avatar sale sin marca de agua** y **nada entra ni sale sin pasar el filtro NSFW**. Si una tarea te pide saltarte una de las dos "solo para probar", recházala y anótalo en `MEMORY.md`.

## Estado actual — YA IMPLEMENTADO (2026-07-13)

El mock de Unsplash **ya no existe**. `generate_avatar_background` en `generations.py` llama de verdad a **Pollinations.ai**, aplica un watermark real con Pillow y guarda los avatares en disco. Verificado en vivo: se descargó una imagen real y se le estampó la marca visible.

**Antes de tocar el pipeline, lee el código, no asumas:** `backend/app/services/image_provider.py`, `backend/app/services/watermark.py`, `backend/app/services/security_log.py`, `backend/app/media_paths.py`.

## Proveedor: Pollinations.ai (no Replicate, no Gemini)

Se eligió por ser **gratuito** (pay-as-you-go de Replicate y Gemini quedaron descartados por costo).

- **Endpoint:** `GET https://image.pollinations.ai/prompt/{prompt_url_encoded}`
- **Auth:** opcional — funciona en modo anónimo (verificado), pero si `POLLINATIONS_API_KEY` está en `backend/.env`, se manda como `Authorization: Bearer <key>`. Consíguela en `enter.pollinations.ai`.
- **Parámetros ya fijados** en `image_provider.py`: `model=flux`, `nologo=true` (quita el logo de Pollinations, nosotros ponemos el nuestro), `safe=true`, `private=true`.
- **Respuesta:** bytes crudos de imagen (JPEG), no JSON. `image_provider.py` ya lo maneja.
- **No hay parámetro de variaciones (`n`)**: cada llamada genera una imagen. Para 3–6 variaciones se hacen esa cantidad de llamadas **en paralelo** con `asyncio.gather` (ver `generate_avatar_background`), no secuenciales — así se mantiene la latencia baja.
- **Reintentos:** 3, con backoff exponencial, ya implementados en `image_provider.py`. Un rechazo NSFW **no se reintenta** (es una excepción distinta, `NSFWRejected`).

## Limitación deliberada: sin foto→avatar personalizado (decisión del usuario, 2026-07-13)

Pollinations tiene un modelo `kontext` para imagen→imagen, pero **exige que la foto de entrada ya esté en una URL pública**. El proyecto no tiene storage público (sin S3/CDN en el Alpha), así que **se decidió explícitamente diferir esta función**.

Estado actual: si el usuario sube solo una foto sin prompt, el pipeline genera un avatar del **estilo elegido**, no personalizado a partir de la foto. Esto está documentado en un comentario en `generate_avatar_background` — **no lo quites** sin resolver primero el storage público. Cuando se retome, la tarea es: montar un endpoint que sirva la imagen subida en una URL alcanzable (o usar un storage público real) y llamar a `POST /v1/images/edits` de Pollinations con esa URL.

## Filtro NSFW — estado real (corregido 2026-07-14, no confíes en versiones anteriores de esta sección)

- **Entrada: DESACTIVADO, no cubierto.** El backlog (`B-03`) documenta que `safe=true` se puso en `safe=false` "temporalmente para testing" el 2026-07-13 porque el filtro de Pollinations era demasiado estricto. **Ese cambio sigue en el código.** Ningún prompt de usuario pasa por moderación de proveedor hoy. Esto es un incumplimiento directo de `SOUL.md §4` ("toda imagen y todo prompt pasa por filtro NSFW... en entrada"), y la tarea sigue marcada `[x]` en el backlog por error — no lo repitas sin verificar `image_provider.py` primero.
- **Salida: implementado con NudeNet (`nsfw_filter.py::validate_image_content`), pero sin verificar en la práctica.** El código en modo `"moderate"` solo compara contra `explicit_categories` (`EXPOSED_ANUS`, `EXPOSED_BREAST_F`, `EXPOSED_GENITALIA_F`, `EXPOSED_GENITALIA_M`), que son nombres de una versión antigua de la librería `nudenet`. Existe un segundo set, `suggestive_categories`, con los nombres de la API nueva (`FEMALE_BREAST_EXPOSED`, etc.) que **nunca se usa en modo moderate**. Si la versión de `nudenet` instalada (ver `requirements.txt`) emite las etiquetas nuevas, el filtro de salida no rechaza nada y `validate_image_content` devuelve `True` siempre. **No asumas que funciona porque no lanza error** — antes de tocar este archivo, corre `detector.detect()` contra una imagen de fixture explícita y mira qué `class` devuelve de verdad.
- Todo rechazo de entrada se registra con `log_nsfw_rejection()` en `backend/app/media/security.log` (JSONL, append-only, sin endpoint de borrado) — pero mientras `safe=false` esté activo, no habrá rechazos de entrada que registrar.
- El rechazo llega al frontend como un evento `generation_failed` normal por WebSocket, con un campo extra `error_code: "GEN_004"` — **no como un 422 HTTP**, porque el POST inicial ya respondió 202 antes de que se sepa si hay contenido rechazado.

**Prioridad si tomas esta skill:** reactivar `safe=true` (o confirmar por qué sigue en `false` y documentarlo como decisión, no como olvido) y escribir el test de fixture para las categorías de NudeNet. Ambas cosas son más urgentes que B-09.

## Watermark — implementado

`services/watermark.py::apply_watermark()` usa Pillow para estampar el texto **"ProyectoIA · Alpha"** sobre los píxeles, esquina inferior derecha, con sombra para legibilidad sobre cualquier fondo. Se aplica a **toda** imagen antes de guardarla, sin excepción de plan. Verificado visualmente.

## Almacenamiento — sin S3/CDN, en disco local

Los avatares se guardan en `backend/app/media/avatars/` y se sirven vía `StaticFiles` montado en `/media` (`main.py`). `cdn_url` en la base de datos es una ruta relativa (`/media/avatars/{id}.png`), no una URL absoluta — funciona porque Vite ya proxea `/media` a `localhost:8000` (`vite.config.ts`), igual que `/api`. Si despliegan a producción sin ese mismo patrón de proxy/reverse-proxy compartiendo origen, las URLs de avatares se romperán.

## Cómo verificar tu trabajo

```bash
cd backend
source venv/Scripts/activate   # o venv/bin/activate en Linux/Mac
python -c "
import asyncio
from app.services.image_provider import generate_image
asyncio.run(generate_image('test prompt'))
"
```
Para el flujo completo: levanta el backend, dispara una generación real desde el frontend, y **mira la imagen descargada con tus propios ojos** — confirma que es un avatar generado (no una foto de stock) y que la marca de agua es visible.
