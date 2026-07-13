# HEARTBEAT — El Pulso del Proyecto

> **Foto del estado ACTUAL.** Este archivo se **sobrescribe** en cada cambio de turno.
> No es un historial: si buscas qué pasó antes, ve a `MEMORY.md`.

**Última actualización:** 2026-07-13 — por el **Agente Orquestador** (actuando como Agente IA)

---

## Estado Backend — 🟢 Generación real funcionando

- Monolito FastAPI, puerto 8000, sin Celery ni Redis. ✅
- Rutas `/api/v1` sin cambios de contrato: `auth`, `styles`, `generations` (202 + WebSocket + historial).
- **La generación de avatares YA NO es un mock.** `generate_avatar_background` llama de verdad a **Pollinations.ai** (`image.pollinations.ai/prompt/...`), genera las variaciones en paralelo con `asyncio.gather`, les estampa un **watermark real con Pillow** y las guarda en `backend/app/media/avatars/`, servidas por `StaticFiles` en `/media`.
- **Verificado en vivo, no solo "el código se ve bien":** se descargó una imagen real de Pollinations y se confirmó visualmente que el watermark "ProyectoIA · Alpha" queda legible sobre la foto.
- Filtro NSFW de **entrada** activo (`safe=true` de Pollinations, fail-closed). Filtro de **salida** — **no existe todavía**, es la brecha más importante que queda abierta.
- Créditos: se cobran solo al completar, igual que antes. Un rechazo NSFW no cobra.
- Nuevos módulos: `app/media_paths.py`, `app/services/image_provider.py`, `app/services/watermark.py`, `app/services/security_log.py`.

## Estado Frontend — 🟡 Ya estaba cableado (se subestimó en el heartbeat anterior)

Corrección a la entrada previa de este archivo: al auditar de verdad, `Generate.tsx` **ya llamaba** a `POST /api/v1/generations` y ya abría el WebSocket de progreso — no había que cablearlo desde cero. `vite.config.ts` proxea `/api` a `localhost:8000`; se le añadió también el proxy de `/media` para que las descargas de avatares funcionen en dev.

**⚠️ Hallazgo nuevo y peligroso:** `Generate.tsx` tiene un fallback `runSimulation()` que se dispara en `socket.onerror` y **muestra fotos de stock de Unsplash como si fueran el resultado real**, sin avisar al usuario. Con el backend mockeado esto no importaba; ahora que la generación es real, un corte de WebSocket hace que el usuario reciba una imagen falsa sin saberlo. Ver backlog `C-08`.

## Estado IA — 🟢 Integrado con Pollinations.ai (no Replicate, no Gemini)

- Se evaluaron Replicate y Gemini; se eligió **Pollinations.ai** por ser gratuito.
- Funciona sin API key (modo anónimo verificado), pero soporta `POLLINATIONS_API_KEY` en `backend/.env` si se consigue una.
- **Diferido a propósito:** foto→avatar personalizado (`kontext`) porque exige URL pública de la imagen de entrada y no hay storage público en el Alpha. Hoy, subir solo una foto sin prompt genera un avatar del estilo elegido, no personalizado a partir de la foto.

---

## 🎯 Foco Actual

**Agente Backend/IA:** cerrar la brecha de NSFW de salida (`B-04`) — no hay ningún filtro sobre la imagen ya generada, y `SOUL.md §4` exige dos puntos de control.

**Agente Frontend:** arreglar `C-08` (el fallback que muestra imágenes falsas sin avisar) antes de seguir con cualquier otra tarea de UI — es un riesgo de confianza de producto, no una mejora cosmética.

## 🚧 Bloqueos

1. **B-04 (NSFW de salida)** necesita decidir e integrar un servicio de moderación de imágenes; Pollinations no ofrece uno documentado.
2. **B-09 (foto→avatar personalizado)** está bloqueado hasta que exista algún storage con URL pública (aunque sea temporal/dev, tipo ngrok, o ya el storage real post-Alpha).
