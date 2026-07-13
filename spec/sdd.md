# Software Design Document (SDD)
## Sistema de Creación de Identidades Visuales Digitales

> **Versión:** 2.0  
> **Fecha:** 2026-07-13  
> **Estado:** Aprobado (Alpha)  
> **Autores:** Equipo de Arquitectura

> ⚠️ **Nota de versión 2.0:** Esta revisión reemplaza la arquitectura de microservicios de la v1.0 por un **monolito FastAPI**, que es lo que realmente está implementado en `backend/` a la fecha. La v1.0 sobre-diseñó el sistema (microservicios, Celery+Redis, PgBouncer, Kubernetes) antes de tener un Alpha funcionando. Este documento distingue explícitamente **Fase Alpha (actual)** de **Fase Escala (post-Alpha, activada solo cuando haya evidencia de necesitarla)**.

---

## 1. Introducción

### 1.1 Propósito
Este documento describe el diseño técnico completo del **Sistema de Creación de Identidades Visuales Digitales**, una plataforma SaaS impulsada por IA para la generación de avatares profesionales. Sirve como referencia para todos los ingenieros del proyecto.

### 1.2 Alcance
Cubre la arquitectura del sistema completo: frontend, backend, pipeline de IA, almacenamiento, seguridad y estrategia de despliegue.

### 1.3 Documentos Relacionados

| Documento | Ubicación |
|---|---|
| Misión del Sistema | `spec/mision.md` |
| Constitución del Sistema | `spec/constitucion.md` |
| Features - Generación de Avatares | `spec/features/avatar-generation.md` |
| Features - Suscripciones | `spec/features/subscriptions.md` |
| Features - Catálogo de Estilos | `spec/features/style-catalog.md` |
| Features - Gestión de Usuarios | `spec/features/user-management.md` |
| Features - Avatares Corporativos | `spec/features/corporate-avatars.md` |

---

## 2. Arquitectura General

### 2.1 Estilo Arquitectónico (Fase Alpha — ACTUAL)
La plataforma adopta un **monolito modular en FastAPI**. La generación de avatares se ejecuta con `BackgroundTasks` de FastAPI dentro del mismo proceso, y el progreso se notifica al cliente mediante un WebSocket in-process (sin broker externo). No hay microservicios ni cola distribuida en esta fase.

```
┌─────────────────────────────────────────────────────────┐
│                      CLIENTE                            │
│               PWA (React + Vite + TS)                   │
└───────────────────┬─────────────────────────────────────┘
                    │ HTTPS + WebSocket
┌───────────────────▼─────────────────────────────────────┐
│              BACKEND — FastAPI (monolito)                │
│  Routers → Schemas → Services → Repositories → Models    │
│  Auth (JWT HS256) · Rate limiting · CORS                 │
│  BackgroundTasks: pipeline de generación de avatar        │
│  ConnectionManager: WebSocket progreso in-process          │
└───────────────────┬─────────────────────────────────────┘
                    │
┌───────────────────▼─────────────────────────────────────┐
│                    PostgreSQL 15                          │
│         (única fuente de estado del sistema)               │
└─────────────────────────────────────────────────────────┘
```

Proveedor de IA (Replicate/DALL-E) y Blob Storage (S3/GCS) se integran como llamadas HTTP directas desde el `BackgroundTask`, no como servicios separados. Mientras no haya integración real, el pipeline usa imágenes de placeholder (ver `backend/app/api/v1/generations.py`).

### 2.2 Principios de Diseño (Alpha)
- **Monolito primero:** Un solo proceso FastAPI, un solo `Dockerfile`, una sola base de datos. Nada de infraestructura que no tenga una razón concreta ya presente en el código.
- **No bloqueante donde importa:** La generación de avatar corre en `BackgroundTasks`, no bloquea la respuesta HTTP inicial (202 Accepted).
- **Fail Fast:** Validaciones tempranas (crédito, formato, tamaño, tier) antes de crear el `GenerationRequest`.
- **Observability mínima viable:** Logs a stdout. Prometheus/Grafana/Loki quedan en el roadmap de Fase Escala (§7.4), no son requisito del Alpha.

### 2.3 Fase Escala (post-Alpha, condicional)
Los siguientes componentes **no se implementan hasta que haya evidencia de que el monolito no da abasto** (carga real, tiempos de generación medidos, número de usuarios concurrentes):
- Cola de tareas distribuida (Celery + Redis) si `BackgroundTasks` deja de ser suficiente para el volumen de jobs.
- Separación en servicios (Auth / Core API / AI Generation) si el equipo o el despliegue lo justifican.
- PgBouncer si el pool de conexiones de SQLAlchemy se vuelve el cuello de botella.
- Kubernetes/Terraform si el despliegue con Docker Compose deja de ser manejable.

Cada uno de estos ítems debe activarse con un ADR propio que documente la evidencia que lo motivó (ver §8).

---

## 3. Componentes del Sistema

### 3.1 Frontend — PWA (React + Vite)

| Aspecto | Decisión |
|---|---|
| Framework | React 18 + TypeScript |
| Build tool | Vite 5 |
| Estado global | Zustand |
| HTTP client | TanStack Query (React Query) |
| UI base | Componentes propios + Radix UI primitives |
| Estilos | CSS Modules + Variables CSS |
| Testing | Vitest + Playwright (e2e) |

**Rutas principales:**
```
/                    → Landing page
/app/dashboard       → Panel principal del usuario
/app/generate        → Flujo de generación de avatar
/app/history         → Historial de generaciones
/app/profile         → Configuración de cuenta
/app/billing         → Gestión de suscripción
/admin               → Panel de administración (rol ADMIN+)
```

### 3.2 Backend — FastAPI (monolito)

**Responsabilidades (todas en el mismo proceso FastAPI):**
- CRUD de usuarios y perfiles
- Autenticación (login/registro) y emisión de JWT
- Gestión de créditos y cuotas
- Catálogo de estilos
- Pipeline de generación de avatar, ejecutado vía `BackgroundTasks`
- Notificación de progreso vía WebSocket in-process (`ConnectionManager`)
- Historial de generaciones
- (Roadmap) Webhooks de eventos de suscripción — no implementado en Alpha, ver `spec/mision.md` §Exclusiones

**Stack real:**
- Python 3.11 + FastAPI
- SQLAlchemy 2.0 **síncrono** (`psycopg2-binary`) + Alembic — migrar a async (`asyncpg`) queda en Fase Escala si el I/O de DB se vuelve cuello de botella, no antes.
- Pydantic v2 para validación
- PostgreSQL 15 como base de datos principal

**Estructura de un endpoint (contrato):**
```
Router → Schema (Pydantic v2) → Service → Repository → Model (SQLAlchemy) → DB
                                    └──► BackgroundTask → Proveedor IA (llamada HTTP directa)
```

### 3.3 Pipeline de Generación (dentro del monolito)

**Responsabilidades:**
- Recepción y validación de la solicitud (crédito, formato, tamaño, tier, estilo)
- Ejecución del pipeline como `BackgroundTask` (no bloquea la respuesta 202)
- Pre-procesado de imagen de entrada (resize, normalización) — pendiente de implementar
- Orquestación de la llamada al modelo de IA — pendiente; actualmente usa imágenes de placeholder
- Post-procesado (upscaling, watermark para free tier) — pendiente
- Almacenamiento de resultados en Blob Storage — pendiente; actualmente usa URLs externas de placeholder
- Emisión de progreso/resultado vía WebSocket in-process

**Stack real:**
- Mismo proceso FastAPI que el resto del backend (sin Celery ni Redis)
- Integración prevista con Replicate / DALL-E 3 (pendiente, ver `spec/roadmap.md`)
- Pillow / OpenCV para pre y post-procesado (pendiente)

> ⚠️ El pipeline actual **no genera imágenes reales ni aplica filtro NSFW** — usa placeholders de Unsplash. Antes de conectar un proveedor de IA real, el filtro NSFW (Constitución §2.3) es un bloqueante obligatorio, no opcional.

### 3.4 Autenticación

**Mecanismo real:** JWT firmado con **HS256** (access token 30 min, ver `backend/app/config/settings.py`). Refresh token de 7 días definido en `auth_handler.py`, pero **la rotación con detección de reutilización aún no está implementada**.
**Proveedores:** Email/Password local. OAuth2 (Google, GitHub) es roadmap, no está implementado.
**Librería:** `python-jose` + `bcrypt` (cost 12)

> La migración de HS256 a RS256 (ADR-0005 histórico) queda pospuesta a Fase Escala: solo aporta valor real cuando haya más de un servicio verificando tokens. Con un monolito único, HS256 es suficiente y más simple.

---

## 4. Modelo de Datos

### 4.1 Entidades Principales

```sql
-- Usuarios
users
  id            UUID PK
  email         VARCHAR(255) UNIQUE NOT NULL
  display_name  VARCHAR(100)
  avatar_url    TEXT
  plan_tier     ENUM('free', 'pro', 'enterprise') DEFAULT 'free'
  credits_used  INTEGER DEFAULT 0
  credits_limit INTEGER DEFAULT 5
  created_at    TIMESTAMP
  updated_at    TIMESTAMP

-- Solicitudes de generación
generation_requests
  id            UUID PK
  user_id       UUID FK → users.id
  style_id      UUID FK → styles.id
  prompt        TEXT
  input_image_url TEXT
  status        ENUM('pending','processing','completed','failed')
  variations    INTEGER  -- cantidad de avatares solicitados (3-6)
  created_at    TIMESTAMP
  completed_at  TIMESTAMP

-- Avatares generados
generated_avatars
  id            UUID PK
  request_id    UUID FK → generation_requests.id
  storage_key   TEXT NOT NULL  -- Clave en S3/GCS
  cdn_url       TEXT
  resolution    VARCHAR(20)  -- "512x512", "1024x1024"
  is_premium    BOOLEAN DEFAULT FALSE
  expires_at    TIMESTAMP  -- NULL = permanente (premium)
  created_at    TIMESTAMP

-- Catálogo de estilos
styles
  id            UUID PK
  name          VARCHAR(100)
  slug          VARCHAR(100) UNIQUE
  category      ENUM('professional','gaming','corporate','social','education','marketing')
  description   TEXT
  preview_url   TEXT
  base_prompt   TEXT  -- Prompt base del estilo (encriptado)
  is_active     BOOLEAN DEFAULT TRUE
  tier_required ENUM('free','pro','enterprise') DEFAULT 'free'

-- Suscripciones
subscriptions
  id            UUID PK
  user_id       UUID FK → users.id
  plan          ENUM('pro','enterprise')
  status        ENUM('active','cancelled','past_due','trialing')
  provider      ENUM('stripe','paypal')
  external_id   VARCHAR(255)  -- ID en Stripe/PayPal
  current_period_start TIMESTAMP
  current_period_end   TIMESTAMP
  created_at    TIMESTAMP
```

---

## 5. Pipeline de Generación de Avatar

```
1. Usuario envía solicitud (foto o prompt + estilo)         [implementado]
        ↓
2. API valida: formato, tamaño, cuota de créditos, tier      [implementado]
        ↓
3. API crea GenerationRequest (status=pending) y responde     [implementado]
   202 con request_id; encola BackgroundTask (mismo proceso)
        ↓
4. BackgroundTask pasa status a 'processing' y notifica        [implementado]
   progreso por WebSocket
        ↓
5. Pre-procesado: resize a 512px, RGB, eliminación EXIF        [pendiente]
        ↓
6. Construcción del prompt: base_prompt(estilo) + prompt_usuario [pendiente]
        ↓
7. Llamada al modelo de IA (Replicate / DALL-E API)             [pendiente — hoy usa placeholders]
        ↓
8. Filtro NSFW en salida (bloqueante antes de servir imagen real) [pendiente — obligatorio antes de 7]
        ↓
9. Post-procesado: upscaling, watermark si es free tier          [pendiente]
        ↓
10. Upload a Blob Storage, generación de URL CDN firmada          [pendiente — hoy URLs externas fijas]
        ↓
11. Actualización de estado en DB → 'completed' + notificación   [implementado]
    WebSocket al cliente
        ↓
12. Decremento de crédito del usuario                             [implementado]
        ↓
13. Limpieza de imagen de entrada (después de 24h)                [pendiente]
```

Los pasos marcados `[pendiente]` son el trabajo real que falta para pasar de Alpha (placeholders) a un pipeline de IA funcional, y deben priorizarse en ese orden en `spec/roadmap.md`.

---

## 6. Seguridad

### 6.1 Autenticación y Autorización
**Estado actual:** JWT firmado con HS256, `SECRET_KEY` en `.env` (`backend/app/auth/auth_handler.py`). Sin rate limiting implementado todavía.
**Roadmap (bloqueante antes de producción, no antes de Alpha):**
- Rate limiting: 100 req/min por IP, 20 generaciones/min por usuario
- Refresh token rotation con detección de reutilización
- RS256 solo si/cuando se separen servicios (§2.3)

### 6.2 Protección de Contenido
**Estado actual:** No implementado — el pipeline usa placeholders, no genera imágenes reales todavía.
**Bloqueante antes de conectar un proveedor de IA real (no antes, no después):**
- Filtro NSFW en imágenes de entrada (pre-procesado) y salida (post-procesado)
- Proveedor: SafeSearch API de Google Cloud Vision o modelo local NudeNet
- Contenido flaggeado: bloqueo inmediato + log inmutable + posible ban de cuenta

Este es el único punto de la Constitución (§2.3) que no admite excepción de "simplificar para el Alpha": si se conecta un proveedor de IA real sin este filtro, no se despliega.

### 6.3 Almacenamiento Seguro
**Estado actual:** No hay Blob Storage real; `cdn_url` apunta a imágenes de placeholder externas.
**Roadmap al integrar Blob Storage real:**
- Imágenes de entrada en bucket privado (sin acceso público directo)
- Avatares generados accesibles vía URLs pre-firmadas con TTL
- Encriptación at-rest: AES-256 (gestionada por el proveedor cloud)
- Encriptación in-transit: TLS 1.3 obligatorio

---

## 7. Estrategia de Despliegue

### 7.1 Entornos

| Entorno | Propósito | Rama Git |
|---|---|---|
| `development` | Desarrollo local | `feature/*` |
| `staging` | QA y validación | `develop` |
| `production` | Usuarios reales | `main` |

### 7.2 Infraestructura — Alpha (ACTUAL)
- **Contenedores:** un `Dockerfile` para `backend/` (FastAPI + uvicorn) y uno para `frontend/` (build estático). Sin Docker Compose todavía — se agrega solo cuando se necesite levantar Postgres + backend + frontend juntos en un comando.
- **CI/CD:** por definir (no implementado).
- **Base de datos:** PostgreSQL 15 gestionado (managed service), sin pooling adicional.
- **CDN:** ninguno; el frontend se sirve directo hasta que el tráfico lo justifique.

### 7.3 Infraestructura — Fase Escala (post-Alpha, condicional)
Activar solo con evidencia de necesidad, cada uno con su propio ADR:
- Docker Compose / Kubernetes si el equipo crece o el despliegue manual deja de ser viable.
- Terraform si hay más de un entorno cloud que gestionar a mano.
- Prometheus + Grafana + Loki cuando haya suficiente tráfico para que la observabilidad manual (logs) no alcance.
- PgBouncer si el pool de conexiones se agota.
- Cloudflare/CDN cuando el volumen de assets estáticos lo justifique.

### 7.4 Escalabilidad
En Alpha, la escalabilidad no es un objetivo de diseño — lo es la velocidad para validar el producto. Cuando `BackgroundTasks` deje de ser suficiente (jobs que se acumulan, timeouts), la primera palanca es Celery + Redis (§2.3), no antes.

---

## 8. Decisiones de Arquitectura Registradas (ADR)

| ID | Decisión | Estado |
|---|---|---|
| ADR-0001 | Usar FastAPI sobre Django REST Framework | Aceptado |
| ADR-0002 | ~~Cola asíncrona Celery + Redis para generación IA~~ → Reemplazado por `BackgroundTasks` de FastAPI en Alpha | Superado por ADR-0006 |
| ADR-0003 | PostgreSQL como base de datos principal | Aceptado |
| ADR-0004 | Integración con Replicate como proveedor primario de IA | En revisión (aún no implementado) |
| ADR-0005 | ~~JWT con RS256 en lugar de HS256~~ → HS256 en Alpha, RS256 diferido a Fase Escala | Superado por ADR-0007 |
| ADR-0006 | Monolito FastAPI con `BackgroundTasks` en vez de microservicios + Celery/Redis para el Alpha | Aceptado |
| ADR-0007 | JWT HS256 en Alpha (un solo servicio verificando tokens); RS256 se evalúa solo al separar servicios | Aceptado |

*Los ADR detallados se documentan en `spec/adr/` (crear el archivo individual para ADR-0006 y ADR-0007 siguiendo la plantilla de `agent.md` §documentation).*
