# SOUL — Reglas Inmutables del Sistema de Avatares

> **Este archivo NO se modifica durante el desarrollo.** Es el contrato de negocio del Alpha.
> Todo agente lo lee al iniciar. Una tarea que exija violar una de estas reglas se rechaza,
> y el conflicto se anota en `MEMORY.md` para que un humano lo resuelva.

---

## 1. Producto

**Qué es:** una plataforma web que convierte una **foto** o una **descripción textual** en avatares profesionales generados con IA, en menos de 60 segundos.

**Qué NO es en el Alpha:** no hay video, ni animación, ni editor de imágenes, ni app móvil nativa, ni voz, ni moderación humana, ni pasarela de pagos real.

**Entrada válida:** imagen JPEG/PNG/WEBP de ≤ 10 MB, **o** un prompt de texto de 10 a 500 caracteres. Al menos uno de los dos es obligatorio.

**Salida:** de 3 a 6 variaciones por solicitud, PNG de 512×512.

---

## 2. Roles de Negocio (Planes)

| Plan | Créditos/mes | Estilos | Watermark | Resolución | Retención historial |
|---|---|---|---|---|---|
| **free** | 5 | solo estilos `tier_required = free` | **SÍ, siempre** | 512×512 | 30 días |
| **pro** | 100 | free + pro | No | hasta 1024×1024 | permanente |
| **enterprise** | ilimitado (negociado) | todos, incl. exclusivos | No | hasta 1024×1024 | permanente |

Reglas duras de plan:
- **Un crédito se descuenta solo cuando la generación se COMPLETA.** Una generación fallida **nunca** cobra crédito.
- Con 0 créditos disponibles, el endpoint de generación responde **403** con código `GEN_003` y el UI deshabilita el botón mostrando un CTA de upgrade.
- Un usuario `free` que pida un estilo `pro` recibe **403**. Nunca se degrada silenciosamente a otro estilo.
- El plan vive en `users.plan_tier`. La cuota vive en `users.credits_used` / `users.credits_limit`. **No dupliques esta verdad en otro sitio.**

---

## 3. Marca de Agua (Watermark) — NO NEGOCIABLE EN EL ALPHA

**Todo avatar generado en el Alpha lleva marca de agua**, sin excepción, sea cual sea el plan del usuario.

- Motivo: el Alpha no tiene pagos reales; sin watermark universal no hay forma de proteger la salida.
- El campo `GeneratedAvatar.is_watermarked` **siempre es `True`** mientras dure el Alpha.
- El watermark se aplica en **post-procesado del backend**, nunca en CSS del frontend. Un watermark que se quita con DevTools no es un watermark.
- Cuando el Alpha termine y existan pagos, esta regla se relaja a: watermark solo para `free`. **Ese cambio requiere aprobación humana explícita**, no lo decide ningún agente.

---

## 4. Filtro NSFW — NO NEGOCIABLE

**Toda imagen y todo prompt pasa por filtro NSFW automático**, en dos puntos:

1. **Entrada:** la foto subida y el prompt de texto se validan ANTES de llamar al modelo de IA.
2. **Salida:** la imagen generada se valida ANTES de guardarla y mostrarla al usuario.

Reglas duras:
- Contenido rechazado → **422** con código `GEN_004` y mensaje neutro al usuario. **No** se describe qué se detectó ni se devuelve la imagen ofensiva.
- Un rechazo NSFW **no descuenta crédito**.
- Todo rechazo se registra en un log de seguridad **inmutable** (append-only): timestamp, `user_id`, tipo de entrada, motivo. Ningún rol de la aplicación puede borrar ese log.
- **El fallo del filtro se resuelve rechazando.** Si el servicio de moderación no responde, la generación se rechaza; nunca se deja pasar contenido sin validar.

---

## 5. Privacidad de los Datos del Usuario

- La **imagen de entrada** subida por el usuario se elimina a las **24 horas**. Es material sensible: no se conserva "por si acaso".
- Se hace **strip de metadatos EXIF** a toda imagen subida (contiene geolocalización).
- El usuario puede exportar sus datos y eliminar su cuenta; eliminar la cuenta borra imágenes y avatares del almacenamiento, no solo la fila en la base de datos.
- **Nunca** se registran en logs: contraseñas, tokens JWT, ni el contenido binario de las imágenes.
- Las claves de API (proveedor de IA, base de datos, `SECRET_KEY`) viven **solo** en variables de entorno. Jamás se escriben en el código ni se commitean.

---

## 6. Arquitectura del Alpha (Restricción Técnica Inmutable)

- **Monolito FastAPI.** Un solo servicio, un solo despliegue.
- **PROHIBIDO en el Alpha:** Celery, Redis, RabbitMQ, microservicios, PgBouncer, Kubernetes, service mesh.
- El trabajo asíncrono se hace con **`BackgroundTasks` de FastAPI**. Nada más.
- Stack fijado: **Python 3.11 + FastAPI + SQLAlchemy + PostgreSQL** / **React + TypeScript + Vite**.
- Si una solución "correcta a futuro" exige infraestructura nueva, se elige la simple y se anota la deuda técnica en `MEMORY.md`.
