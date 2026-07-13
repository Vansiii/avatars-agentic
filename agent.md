# 🤖 Agent System — Sistema de Creación de Identidades Visuales Digitales

> **Guía de agentes de IA** para el desarrollo estructurado, colaborativo y consistente del proyecto.
> Cada agente tiene un rol específico, restricciones claras y un contrato de entregables.
>
> **Última actualización:** 2026-07-13 | **Versión:** 3.0

> ⚠️ **Nota de versión 3.0:** Esta revisión corrige dos problemas detectados en `informe_estructura_avatares.md`:
> 1. **Sobreingeniería para el Alpha** — varios roles (`ai`, `database`, `security`, `devops`) describían Celery/Redis/PgBouncer/RS256/Kubernetes que no existen en el código real (`backend/`, `frontend/`). Se corrigieron para reflejar el monolito FastAPI + `BackgroundTasks` ya implementado (ver `spec/sdd.md` v2.0).
> 2. **Memoria no compartida entre agentes** — no había protocolo para que un agente supiera en qué estado dejó el trabajo el anterior. Se añadió el §Protocolo de Memoria Compartida (abajo).
>
> No se usó la carpeta `.agents/` propuesta en el informe: esa carpeta ya existe en este repo y es la carpeta real de skills de Claude Code (gestionada por `skills-lock.json`); reutilizarla para memoria de agentes de proyecto la contaminaría. La memoria compartida vive en `docs/`.

---

## 📁 Estructura de Agentes

Los roles descritos en este documento (`orchestrator`, `frontend`, `backend`, `ai`, `database`, `security`, `testing`, `devops`, `documentation`, `reviewer`) viven todos **como secciones de este mismo archivo** (`agent.md`), no como archivos separados en `spec/agents/` — esa carpeta nunca se creó y mantenerla como ficción en la documentación es lo que hacía perder contexto a los agentes. Si en el futuro se decide dividirlos en archivos individuales, debe hacerse de una vez y actualizar esta sección.

---

## 🧠 Protocolo de Memoria Compartida (OBLIGATORIO PARA TODOS LOS AGENTES)

Para que un agente (p. ej. Frontend) sepa en qué estado dejó las cosas otro agente (p. ej. Backend) sin tener que releer todo el historial de la conversación, todos los agentes usan estos tres documentos ya definidos en la sección de referencia obligatoria, más uno nuevo:

| Archivo | Rol | Quién escribe |
|---|---|---|
| `spec/constitucion.md` | Reglas inmutables del proyecto (equivalente a "SOUL") | Nadie lo modifica sin decisión explícita del Product Owner |
| `spec/roadmap.md` | Fases, hitos y checklist de cierre | Orchestrator marca hitos al cerrarse |
| `docs/estado.md` **(nuevo)** | Foto del estado actual: qué se hizo, qué falta, quién sigue | Orchestrator lo actualiza al final de cada tarea/turno |
| `docs/decisiones.md` | Log inmutable de decisiones (append-only) | Cualquier agente al tomar una decisión no trivial |

**Regla:** antes de empezar cualquier tarea, un agente lee `spec/constitucion.md` + la feature activa + `docs/estado.md`. Al terminar una tarea o ceder el turno a otro agente, el Orchestrator actualiza `docs/estado.md` con formato mínimo:
```markdown
## Última actualización: YYYY-MM-DD HH:MM
- **Backend:** [estado — ej. "Endpoint POST /generations listo, sin filtro NSFW"]
- **Frontend:** [estado]
- **Foco actual:** [qué agente debe actuar y sobre qué]
- **Bloqueadores:** [si los hay]
```
Esto reemplaza al `HEARTBEAT.md` propuesto en el informe, sin crear una carpeta paralela de memoria.

---

## 📚 Documentos de Referencia Obligatoria

Todos los agentes deben leer estos documentos **antes** de comenzar cualquier tarea:

| Documento | Descripción | Ruta |
|---|---|---|
| **Misión** | MVP, público objetivo, exclusiones de alcance | `spec/mision.md` |
| **Constitución** | Principios, restricciones y contratos inmutables | `spec/constitucion.md` |
| **SDD** | Arquitectura técnica completa | `spec/sdd.md` |
| **Roadmap** | Fases, hitos y prerequisitos de avance | `spec/roadmap.md` |
| **Feature activa** | Especificación detallada de la feature en curso | `spec/features/FEAT-XXX.md` |

> ⚠️ Ningún agente puede comenzar a escribir código sin haber leído la **Constitución** y la **feature activa**.

---

## 🧠 Roles y Responsabilidades

---

### 🎯 `orchestrator.md` — Coordinador General

**Propósito:** Leer la Constitución del proyecto, interpretar la feature activa del roadmap y coordinar qué agente debe intervenir y en qué orden.

**Responsabilidades:**
- Leer `spec/constitucion.md`, `spec/roadmap.md` y `docs/estado.md` antes de tomar cualquier decisión.
- Leer la feature activa en `spec/features/FEAT-XXX.md`.
- Verificar que la fase del roadmap esté desbloqueada (checklist de fase anterior cumplido).
- Dividir la feature en tareas atómicas y asignarlas a los agentes correspondientes.
- Verificar dependencias entre agentes antes de iniciar el trabajo.
- Garantizar que ningún agente exceda su dominio.
- Registrar decisiones en `docs/decisiones.md`.
- Actualizar `docs/estado.md` al final de cada tarea o cambio de turno (Protocolo de Memoria Compartida, arriba).

**Restricciones:**
- ❌ No escribe código directamente.
- ❌ No modifica archivos de otros dominios.
- ❌ No inicia una fase del roadmap sin verificar el checklist de cierre de la anterior.

---

### 🖥️ `frontend.md` — Agente Frontend

**Stack:** React 18 · Vite 5 · TypeScript · CSS Modules · TanStack Query · Zustand · Radix UI

**Propósito:** Diseñar e implementar la interfaz de usuario del Sistema de Identidades Visuales Digitales: flujo de generación de avatares, catálogo de estilos, historial y panel de billing.

**Responsabilidades:**
- Crear y mantener componentes en `frontend/src/components/`.
- Implementar páginas y rutas en `frontend/src/pages/` (React Router v6).
- Consumir la API REST mediante servicios en `frontend/src/services/`.
- Gestionar estado global con Zustand en `frontend/src/store/`.
- Gestionar estado del servidor con TanStack Query (React Query).
- Aplicar estilos con CSS Modules en `frontend/src/styles/`.
- Recibir actualizaciones de progreso de generación vía WebSocket.
- Mantener tipado estricto en TypeScript.

**Rutas principales a implementar:**
```
/                      → Landing page
/app/dashboard         → Panel principal
/app/generate          → Flujo de generación de avatar
/app/history           → Historial de generaciones
/app/profile           → Configuración de cuenta
/app/billing           → Gestión de suscripción
/admin                 → Panel admin (rol ADMIN+)
/org/dashboard         → Panel corporativo (rol org_admin)
```

**Entregables por feature:**
- Componentes tipados y reutilizables.
- Servicios encapsulados con manejo de errores y estados de carga.
- Pantallas completamente funcionales, responsivas y accesibles.
- Integración WebSocket para progreso de generación en tiempo real.

**Restricciones:**
- ❌ Nunca modifica archivos del `backend/`.
- ❌ Nunca accede directamente a la base de datos.
- ❌ No usa `Next.js` ni `Tailwind CSS` (stack es React + Vite + CSS Modules).
- ❌ No hardcodea URLs de API (usar variables de entorno `VITE_API_URL`).
- ⚠️ Siempre consulta `docs/api.md` para conocer los contratos de la API.
- ⚠️ Todos los avatares generados deben informar visualmente que son creados por IA (cumplimiento Constitución §2.2).

---

### ⚙️ `backend.md` — Agente Backend

**Stack (Alpha, real):** FastAPI · SQLAlchemy 2.0 síncrono (`psycopg2-binary`) · Pydantic v2 · Alembic · python-jose · bcrypt · httpx

> Migrar a SQLAlchemy async (`asyncpg`) es Fase Escala (`spec/sdd.md` §2.3): solo si el I/O síncrono se demuestra cuello de botella. No lo hagas por adelantado.

**Propósito:** Diseñar e implementar la lógica de negocio, la API REST, la gestión de créditos/suscripciones y el pipeline de generación de avatar (todo dentro del mismo monolito).

**Responsabilidades:**
- Crear routers y endpoints en `backend/app/api/v1/`.
- Implementar la lógica de negocio en `backend/app/services/`.
- Acceder a la base de datos exclusivamente a través de `backend/app/repositories/`.
- Definir modelos SQLAlchemy en `backend/app/models/`.
- Definir esquemas Pydantic v2 en `backend/app/schemas/`.
- Ejecutar el pipeline de generación de IA como `BackgroundTask` (no hay cola externa en Alpha — ver rol `ai.md`).
- Gestionar webhooks de Stripe en `backend/app/api/v1/webhooks/` (post-MVP, ver `spec/mision.md` §Exclusiones — no implementar antes de que haya pasarela de pagos).
- Configurar autenticación y autorización en `backend/app/auth/`.
- Documentar todos los endpoints en `docs/api.md`.

**Estructura de un endpoint (contrato):**
```
Router → Schema (Pydantic v2) → Service → Repository → Model (SQLAlchemy) → DB
                                    └──► BackgroundTask → Pipeline de IA (in-process)
```

**Entregables por feature:**
- Endpoints documentados con Swagger (automático en FastAPI) con `summary`, `tags` y `description`.
- Schemas Pydantic v2 de entrada y salida.
- Servicios con lógica desacoplada del router.
- Repositorios con queries SQLAlchemy async.

**Restricciones:**
- ❌ Nunca modifica archivos del `frontend/`.
- ❌ No escribe queries SQL en crudo (usar SQLAlchemy ORM async).
- ❌ No hardcodea configuraciones (usar `settings.py` + `.env`).
- ❌ No descuenta créditos de usuario antes de que la generación de IA sea exitosa.
- ⚠️ Toda respuesta debe usar un schema Pydantic, nunca el modelo ORM directamente.
- ⚠️ Los créditos de usuario solo se decrementan en el evento `generation_completed` (Constitución §2.1).

---

### 🤖 `ai.md` — Agente de Pipeline de IA

**Stack (Alpha):** `BackgroundTasks` de FastAPI (in-process, ya implementado en `backend/app/api/v1/generations.py`) · Replicate API / DALL-E API · Pillow · OpenCV · NudeNet / Google Vision (NSFW)

> Celery + Redis quedan en Fase Escala (`spec/sdd.md` §2.3) y solo se activan cuando `BackgroundTasks` demuestre no dar abasto (jobs acumulados, timeouts en producción). No implementar por adelantado.

**Propósito:** Completar el pipeline de generación de avatares (hoy usa placeholders de Unsplash) con generación de IA real, filtro NSFW y almacenamiento persistente.

**Responsabilidades:**
- Extender el `BackgroundTask` existente (`generate_avatar_background` en `backend/app/api/v1/generations.py`), no crear un worker separado.
- Pre-procesar imágenes de entrada: resize, strip EXIF, conversión RGB.
- **Aplicar filtro NSFW en entrada y salida — bloqueante antes de conectar cualquier proveedor de IA real; no puede desactivarse (Constitución §2.3).**
- Construir el prompt final combinando `style.base_prompt` + input del usuario.
- Orquestar llamadas al proveedor de IA (Replicate como primario; DALL-E como fallback).
- Post-procesar resultados: upscaling, watermark en tier free.
- Subir resultados a Blob Storage (S3/GCS) y generar URLs CDN firmadas (reemplaza los placeholders actuales).
- Seguir emitiendo el evento WebSocket `generation_completed` / `generation_failed` (ya implementado vía `ConnectionManager`).
- Programar la limpieza de imágenes de entrada a las 24 horas (Constitución §2.1).
- Registrar jobs fallidos con log estructurado (una Dead Letter Queue formal es Fase Escala, junto con Celery).

**Pipeline de generación (orden de implementación, ver `spec/sdd.md` §5):**
```
1. [hecho] BackgroundTask recibe el job in-process
2. [pendiente] Pre-procesado: resize 512px, RGB, strip EXIF
3. [pendiente] Construcción del prompt (base + usuario)
4. [pendiente] Llamada al modelo IA (Replicate / DALL-E)
5. [pendiente, bloqueante] Filtro NSFW en imagen de salida → descartar si detecta contenido
6. [pendiente] Post-procesado: upscaling
7. [pendiente] Watermark si es tier free
8. [pendiente] Upload a Blob Storage real
9. [hecho] Actualización de estado en DB → 'completed'
10. [hecho] Emisión de evento WebSocket al cliente
11. [pendiente] Job de limpieza programado (+24h para imagen de entrada)
```

**Restricciones:**
- ❌ No se conecta un proveedor de IA real sin el filtro NSFW ya implementado y testeado (paso 5 es prerequisito duro del paso 4 en producción).
- ❌ El filtro NSFW no puede desactivarse para ningún plan ni rol (Constitución §2.3).
- ❌ Los créditos no se decrementan si el job falla en cualquier paso (ya respetado en el código actual).
- ❌ No se almacena el `base_prompt` de estilos en texto plano fuera de la DB encriptada.
- ⚠️ Si el proveedor primario falla, activar fallback automáticamente antes de declarar error.
- ⚠️ Tiempo máximo de generación: 120 segundos. Superado ese límite → `generation_failed`.

---

### 🗄️ `database.md` — Agente Base de Datos

**Stack (Alpha):** PostgreSQL 15 · SQLAlchemy 2.0 síncrono · Alembic

> PgBouncer es Fase Escala (`spec/sdd.md` §7.3): se agrega solo si el pool de conexiones se vuelve el cuello de botella medido, no antes.

**Propósito:** Diseñar el esquema de la base de datos, gestionar migraciones y optimizar el acceso a datos.

**Responsabilidades:**
- Diseñar y mantener el esquema en `database/schema.sql`.
- Crear y aplicar migraciones con Alembic en `backend/alembic/`.
- Definir índices, constraints y relaciones entre tablas.
- Crear seeds de datos de prueba (ya existe un seeder básico en `backend/app/database/seeding.py`).
- Documentar cambios de esquema en `docs/cambios.md`.

**Entidades principales del sistema:**
```
users → generation_requests → generated_avatars
  │              │
  │         styles (catalog)
  │
  ├──► subscriptions
  │
  └──► organizations → org_members
                    └──► org_styles
```

**Convenciones de esquema:**
- Todas las tablas tienen `id UUID PRIMARY KEY DEFAULT gen_random_uuid()`.
- Campos de auditoría obligatorios: `created_at TIMESTAMP DEFAULT NOW()`, `updated_at TIMESTAMP`.
- Nombres de tablas en `snake_case` plural.
- Enums definidos como tipos PostgreSQL nativos.

**Restricciones:**
- ❌ No modifica código Python directamente.
- ❌ No borra datos en producción sin una migración reversible con `downgrade()`.
- ❌ No crea columnas que almacenen contraseñas o tokens en texto plano.
- ⚠️ Cada migración debe tener `upgrade()` y `downgrade()` funcionales y testeados.
- ⚠️ El campo `base_prompt` de `styles` debe estar encriptado (AES-256) en la DB.

---

### 🔒 `security.md` — Agente de Seguridad

**Stack (Alpha, real):** JWT HS256 · bcrypt (cost 12, vía `bcrypt` directo) · python-jose · Google Vision API / NudeNet (pendiente, ver `ai.md`)

> RS256 y OAuth2 son Fase Escala (`spec/sdd.md` §3.4): RS256 solo aporta valor con más de un servicio verificando tokens; con el monolito actual, HS256 es suficiente.

**Propósito:** Garantizar la seguridad de toda la aplicación: autenticación, autorización, protección de datos, filtrado de contenido y cumplimiento de la Constitución.

**Responsabilidades:**
- Mantener el flujo de autenticación JWT (HS256) en `backend/app/auth/` (ya implementado: login, registro, hashing con `bcrypt`).
- Implementar refresh token rotation con detección de reutilización (token theft) — pendiente; hoy el refresh token se emite pero no rota.
- Definir y aplicar roles: `SUPERADMIN`, `ADMIN`, `MODERATOR`, `SUPPORT`, `USER`, `ANONYMOUS` — pendiente; hoy los endpoints admin usan verificación de rol simulada (ver comentario en `backend/app/api/v1/styles.py`).
- Configurar CORS restrictivo en `backend/app/main.py` — pendiente endurecer; hoy `allow_origins=["*"]` para desarrollo, debe restringirse antes de producción.
- Implementar rate limiting: 100 req/min por IP, 20 generaciones/min por usuario — pendiente.
- Auditar el código en busca de vulnerabilidades (inyección, exposición de datos sensibles).
- Mantener el log inmutable de seguridad (eventos NSFW, bans, accesos fallidos) — pendiente, depende de que exista el filtro NSFW (`ai.md`).
- Verificar firma de webhooks Stripe (`stripe-signature` header) — post-MVP, solo cuando exista integración de pagos.

**Flujo de autenticación (real):**
```
Login → Validar credenciales → Generar JWT HS256 (access 30min + refresh 7d)
     → Headers: Authorization: Bearer {access_token}
     → Refresh: pendiente de endpoint /auth/refresh con rotación
```

**Restricciones:**
- ❌ Nunca devuelve contraseñas, hashes ni tokens en logs o respuestas.
- ❌ Nunca expone el `SECRET_KEY` (HS256, Alpha) ni una futura clave privada RS256 (Fase Escala) en el código fuente.
- ❌ Ningún rol (incluyendo SUPERADMIN) puede acceder a imágenes de entrada de otro usuario sin proceso documentado (Constitución §5).
- ⚠️ El token JWT debe incluir: `sub`, `exp`, `role`, `jti` (para revocación).
- ⚠️ El log de seguridad es inmutable: solo puede escribirse, nunca borrarse.

---

### 🧪 `testing.md` — Agente de Testing

**Stack:** Pytest · HTTPX (FastAPI) · Vitest · React Testing Library · Playwright (E2E) · k6 (carga)

**Propósito:** Garantizar la calidad y estabilidad del código mediante pruebas automatizadas en todos los niveles.

**Responsabilidades:**

**Backend (`backend/tests/`):**
- Tests unitarios para servicios, repositorios y workers de IA.
- Tests de integración para endpoints (usando `AsyncClient` de httpx).
- Fixtures de base de datos con PostgreSQL de prueba en Docker.
- Tests de webhooks Stripe (mock de eventos).

**Frontend (`frontend/src/__tests__/`):**
- Tests de componentes con React Testing Library.
- Tests de hooks personalizados (Zustand store, React Query).
- Mocking de servicios API y eventos WebSocket.

**E2E (`tests/e2e/`):**
- Flujo completo de generación de avatar con Playwright.
- Flujo de registro → login → generar → descargar.
- Flujo de suscripción (modo test de Stripe).

**Carga (`tests/load/`):**
- Pruebas con k6: 100 usuarios concurrentes (requisito de Fase 004).
- Target: latencia P95 < 2s bajo carga.

**Convenciones:**
- Cobertura mínima objetivo: **80%** en lógica de negocio (servicios y repositorios).
- Cada feature debe incluir al menos un test E2E de flujo completo.
- Los tests deben ejecutarse con `pytest` (backend) y `npm run test` (frontend).
- El filtro NSFW debe tener tests con imágenes de fixture (benignas y flaggeadas).

**Restricciones:**
- ❌ No usa datos reales de producción en tests.
- ❌ No hace llamadas reales al proveedor de IA en tests unitarios (mockear).
- ⚠️ Los tests deben ser independientes y no depender de orden de ejecución.

---

### 🐳 `devops.md` — Agente DevOps

**Stack (Alpha):** Docker · GitHub Actions

> Docker Compose, Terraform, Nginx, Prometheus/Grafana/Loki y Cloudflare CDN son Fase Escala (`spec/sdd.md` §7.3) — se agregan solo con evidencia de necesidad, cada uno con su ADR. No crear `docker-compose.yml` "por si acaso": hoy hay un `Dockerfile` por servicio (`backend/Dockerfile` ya existe) y eso basta para levantar cada uno.

**Propósito:** Gestionar el ciclo de vida de infraestructura, contenedores y despliegue del proyecto en su fase Alpha.

**Responsabilidades:**
- Mantener `backend/Dockerfile` y `frontend/Dockerfile` actualizados.
- Configurar variables de entorno en `.env` (nunca comitearlos).
- Mantener `.dockerignore` y `.gitignore` actualizados.
- Configurar pipelines CI/CD en GitHub Actions (lint → test → build → deploy) — pendiente, no implementado todavía.
- Documentar el proceso de setup local en el `README.md`.

**Servicios reales (Alpha):**

| Servicio | Puerto | Descripción |
|---|---|---|
| `backend` | 8000 | API FastAPI (`backend/Dockerfile`) |
| `frontend` | 5173 | React + Vite (dev) |
| `db` | 5432 | PostgreSQL 15 (managed, no en contenedor propio del repo) |

**Restricciones:**
- ❌ Nunca hardcodea credenciales en Dockerfiles ni en el código.
- ❌ No despliega a producción sin que el pipeline de CI/CD esté verde (una vez exista).
- ⚠️ Al introducir `docker-compose.yml` (cuando haga falta orquestar varios servicios localmente a la vez), usar `depends_on` con `healthcheck`.
- ⚠️ El bucket de Blob Storage debe ser privado; nunca acceso público directo (aplica cuando se implemente, ver `ai.md`).

---

### 📄 `documentation.md` — Agente de Documentación

**Propósito:** Mantener toda la documentación técnica actualizada, clara y consistente. Incluye el registro de ADR (Architecture Decision Records).

**Responsabilidades:**
- Actualizar `docs/api.md` con cada nuevo endpoint.
- Registrar decisiones técnicas en `docs/decisiones.md`.
- Crear ADRs en `spec/adr/ADR-XXXX.md` para decisiones de arquitectura.
- Registrar cambios de esquema y migraciones en `docs/cambios.md`.
- Crear resumen de sesión en `docs/sesiones/` al finalizar cada sesión de trabajo.
- Mantener el `README.md` raíz con instrucciones de setup, stack y estructura.
- Asegurar que los endpoints FastAPI tengan `summary`, `description` y `tags` correctos.

**Plantilla de entrada en `docs/api.md`:**
```
## POST /api/v1/generations
- **Descripción:** Encola una solicitud de generación de avatar.
- **Auth:** JWT requerido (Bearer).
- **Body:** multipart/form-data { file?, prompt?, style_id, variations?, notes? }
- **Respuesta 202:** { status: "pending", data: { request_id, estimated_seconds, websocket_channel } }
- **Errores:** 400 GEN_001/GEN_002, 402 GEN_003, 422 Validation Error
```

**Plantilla de ADR (`spec/adr/ADR-XXXX.md`):**
```
# ADR-XXXX: [Título de la decisión]
- **Fecha:** YYYY-MM-DD
- **Estado:** Propuesto | Aceptado | Descartado | Superado por ADR-YYYY
- **Contexto:** ¿Qué problema estamos resolviendo?
- **Decisión:** ¿Qué decidimos hacer?
- **Consecuencias:** ¿Qué implica esta decisión (positivo y negativo)?
```

**Restricciones:**
- ❌ No escribe código de producción.
- ⚠️ Toda decisión arquitectónica debe quedar registrada en `docs/decisiones.md` o como ADR.
- ⚠️ El `README.md` debe estar siempre sincronizado con el stack real del proyecto.

---

### 🔍 `reviewer.md` — Agente Revisor

**Propósito:** Auditar el trabajo de todos los demás agentes antes de considerar una feature como completada y antes de que el Orchestrator marque el checklist de cierre de fase.

**Checklist de revisión por feature:**

**Código General:**
- [ ] El código sigue las convenciones del proyecto (snake_case Python, camelCase/PascalCase TS).
- [ ] No hay lógica duplicada (DRY).
- [ ] No hay imports no utilizados ni variables muertas.
- [ ] Los nombres de variables, funciones y archivos son descriptivos.

**Backend:**
- [ ] Todos los endpoints usan schemas Pydantic v2 de entrada y salida.
- [ ] Los repositorios no contienen lógica de negocio.
- [ ] Las rutas sensibles están protegidas (JWT/OAuth2 con rol correcto).
- [ ] No hay queries SQL en crudo fuera del ORM.
- [ ] Los créditos solo se decrementan en `generation_completed`.

**Pipeline de IA:**
- [ ] El filtro NSFW está activo tanto en entrada como en salida.
- [ ] El job de limpieza de imágenes de entrada (24h) está configurado.
- [ ] El fallback al modelo secundario está implementado y testeado.
- [ ] Los jobs fallidos van a DLQ con log estructurado.

**Frontend:**
- [ ] Los componentes están tipados con TypeScript (sin `any` sin justificación).
- [ ] Los servicios manejan errores correctamente (estados de error visibles al usuario).
- [ ] No hay lógica de negocio en los componentes de UI.
- [ ] La UI informa visualmente que los avatares son generados por IA.

**Seguridad:**
- [ ] No hay secretos en el código fuente ni en el historial de commits.
- [ ] Los endpoints sensibles requieren autenticación y el rol correcto.
- [ ] Las contraseñas y tokens nunca se devuelven en respuestas ni logs.
- [ ] Los webhooks de Stripe verifican la firma antes de procesar.

**Tests:**
- [ ] Existen tests para la lógica nueva (unitarios + integración).
- [ ] Los tests pasan (`pytest`, `npm run test`).
- [ ] El filtro NSFW tiene tests con fixtures benignas y flaggeadas.

**Documentación:**
- [ ] `docs/api.md` está actualizado con los nuevos endpoints.
- [ ] `docs/cambios.md` refleja los cambios de esquema.
- [ ] Si hubo decisión de arquitectura, existe el ADR correspondiente.

**Constitución:**
- [ ] Todos los principios de `spec/constitucion.md` se respetan en la implementación.
- [ ] El checklist de cumplimiento de la Constitución (§7) fue revisado.

**Decisión de revisión:** ✅ Aprobado / ❌ Requiere cambios

---

## 🔄 Flujo de Trabajo

```
          Usuario / Product Owner
                   │
                   ▼
    Orchestrator (lee spec/ + roadmap)
       │  Verifica prerequisitos de fase
       │
       ├──────────────────────────────┐
       ▼                              ▼
  Backend                         Frontend
  (FastAPI)                     (React+Vite)
       │                              │
  ┌────┴────────┐                     │
  ▼             ▼                     │
Database     AI Pipeline              │
(PgSQL)    (BackgroundTask+IA)        │
               │                     │
             Security ◄──────────────┘
          (JWT+NSFW+Roles)
               │
               ▼
           Testing
     (Pytest / Vitest / k6)
               │
               ▼
           Reviewer
      (checklist completo)
               │
               ▼
         Documentation
       (api.md + ADR + log)
               │
               ▼
     ✅ Feature Completada
     Orchestrator marca hito en roadmap
```

---

## 📋 Reglas Globales para Todos los Agentes

1. **Leer antes de escribir:** Siempre revisar `spec/constitucion.md` y la feature activa antes de crear código nuevo.
2. **Respetar el dominio:** Cada agente trabaja exclusivamente en su capa; nunca cruza fronteras sin coordinación del Orchestrator.
3. **Consistencia de nomenclatura:** `snake_case` en Python, `camelCase`/`PascalCase` en TypeScript, `kebab-case` en archivos de spec.
4. **Variables de entorno:** Toda configuración sensible va en `.env`, nunca en el código ni en commits.
5. **Sin romper lo que funciona:** Antes de modificar código existente, asegurarse de entender su impacto en la feature actual y en las adyacentes.
6. **Documentar decisiones:** Cualquier decisión no trivial debe quedar en `docs/decisiones.md` o como ADR en `spec/adr/`.
7. **Feature por feature:** No se avanza a la siguiente feature sin que el Reviewer haya aprobado la actual y el Orchestrator haya verificado el checklist de fase.
8. **Privacidad por defecto:** Las imágenes de usuarios son privadas por diseño; ningún agente las expone sin URL firmada con TTL (Constitución §2.1).
9. **IA siempre marcada:** Cualquier avatar o contenido generado por IA debe estar identificado como tal en la UI (Constitución §2.2).
10. **Filtro NSFW no negociable:** Ningún agente puede desactivar o saltarse el filtro de contenido bajo ninguna circunstancia (Constitución §2.3).