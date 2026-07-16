# Software Design Document (SDD)
## Sistema de Personajes para Spots Publicitarios de TV

> **Versión:** 3.0  
> **Fecha:** 2026-07-14  
> **Estado:** Aprobado (Alpha)

---

## 1. Introducción

### 1.1 Propósito
Este documento describe el diseño técnico del **Sistema de Personajes para Spots Publicitarios de TV**, una plataforma SaaS para crear personajes hiperrealistas y generar videos de spots publicitarios.

### 1.2 Alcance
Frontend, backend, pipeline de generación (imagen + video), seguridad y despliegue.

---

## 2. Arquitectura General

### 2.1 Estilo Arquitectónico (Alpha)
**Monolito modular en FastAPI.** Generación de personajes y videos con `BackgroundTasks`. WebSocket in-process para progreso.

```
┌─────────────────────────────────────────────────────────┐
│                      CLIENTE                            │
│               PWA (React + Vite + TS)                   │
└───────────────────┬─────────────────────────────────────┘
                    │ HTTPS + WebSocket
┌───────────────────▼─────────────────────────────────────┐
│              BACKEND — FastAPI (monolito)                │
│  Routers → Schemas → Services → Models                   │
│  Auth (JWT HS256) · Rate limiting · CORS                 │
│  BackgroundTasks: pipeline personaje + video             │
│  ConnectionManager: WebSocket progreso                   │
└───────────────────┬─────────────────────────────────────┘
                    │
┌───────────────────▼─────────────────────────────────────┐
│                    PostgreSQL 15                          │
└─────────────────────────────────────────────────────────┘
```

### 2.2 Principios de Diseño
- **Monolito primero:** Un solo proceso, un solo Dockerfile, una DB.
- **Fail Fast:** Validaciones tempranas antes de crear registros.
- **Consistencia del personaje:** Imagen de referencia reutilizada en cada generación.

---

## 3. Componentes del Sistema

### 3.1 Frontend — PWA

| Aspecto | Decisión |
|---|---|
| Framework | React 19 + TypeScript |
| Build tool | Vite 5 |
| Estado global | Zustand |
| HTTP client | TanStack Query |
| Estilos | CSS global |

**Rutas principales:**
```
/                    → Landing page
/login               → Login (solo admin crea usuarios)
/app/dashboard       → Panel principal
/app/characters      → Gestión de personajes
/app/characters/new  → Crear personaje
/app/spots           → Gestión de spots
/app/spots/new       → Generar spot
/app/admin/users     → Gestión de usuarios (solo admin)
```

### 3.2 Backend — FastAPI

**Responsabilidades:**
- CRUD de usuarios y autenticación
- CRUD de personajes con gestión de límites
- Pipeline de generación de personajes (imagen)
- Pipeline de generación de spots (video)
- Control de límites semanales
- Sistema de rehacer (3 variaciones, 3 rehacers)

### 3.3 Pipeline de Generación — Personaje

```
1. Recibir entrada (imagen +/o texto)
2. Validar imagen (formato, tamaño, que no sea basura)
3. Si hay imagen: extraer rasgos del rostro
4. Construir prompt hiperrealista
5. Generar 3 variaciones con Pollinations.ai
6. Filtrar NSFW cada variación
7. Retornar para selección del usuario
```

### 3.4 Pipeline de Generación — Spot (Video)

```
1. Recibir: personaje + descripción + tipo
2. Cargar imagen de referencia del personaje
3. Construir prompt de video
4. Generar 3 variaciones de video
5. Filtrar NSFW cada variación
6. Retornar para selección del usuario
```

---

## 4. Modelo de Datos

```sql
-- Usuarios
users
  id            UUID PK
  email         VARCHAR(255) UNIQUE NOT NULL
  display_name  VARCHAR(100)
  role          ENUM('admin', 'user') DEFAULT 'user'
  is_active     BOOLEAN DEFAULT TRUE
  created_at    TIMESTAMP

-- Personajes
characters
  id                  UUID PK
  user_id             UUID FK → users.id
  name                VARCHAR(100) NOT NULL
  description         TEXT
  reference_image_url TEXT          -- imagen de referencia (input del usuario)
  generated_image_url TEXT          -- imagen generada del personaje
  category            VARCHAR(50)   -- deportes, noticias, etc.
  status              ENUM('draft', 'approved', 'active')
  consistency_data    JSONB         -- datos para mantener identidad
  created_at          TIMESTAMP
  updated_at          TIMESTAMP

-- Límites semanales
character_limits
  id              UUID PK
  user_id         UUID FK → users.id
  week_start      DATE NOT NULL
  characters_used INTEGER DEFAULT 0
  spots_used      INTEGER DEFAULT 0
  UNIQUE(user_id, week_start)

-- Spots (videos)
spots
  id              UUID PK
  character_id    UUID FK → characters.id
  user_id         UUID FK → users.id
  script          TEXT NOT NULL
  type            ENUM('short', 'long')
  status          ENUM('pending', 'processing', 'completed', 'failed')
  output_url      TEXT
  duration_seconds INTEGER
  created_at      TIMESTAMP

-- Categorías de spots
spot_categories
  id                    UUID PK
  name                  VARCHAR(100)
  assigned_character_id UUID FK → characters.id
```

---

## 5. Seguridad

### 5.1 Autenticación
JWT firmado con HS256. Admin crea usuarios (no hay registro público).

### 5.2 Protección de Contenido
Filtro NSFW en entrada y salida. Fail-closed.

### 5.3 Límites de Uso
Control semanal por usuario. Verificación en cada endpoint de creación/generación.

---

## 6. Estrategia de Despliegue

### Alpha
- Un Dockerfile para backend, uno para frontend
- PostgreSQL 15 gestionado
- Sin CDN todavía

### Post-Alpha (condicional)
- Docker Compose / Kubernetes
- CDN para assets
- Redis para rate limiting multi-worker

---

## 7. ADRs

| ID | Decisión | Estado |
|---|---|---|
| ADR-0001 | FastAPI sobre Django REST | Aceptado |
| ADR-0003 | PostgreSQL como DB principal | Aceptado |
| ADR-0006 | Monolito con BackgroundTasks | Aceptado |
| ADR-0007 | JWT HS256 en Alpha | Aceptado |
| ADR-0008 | Pollinations.ai para testing gratuito | Aceptado |
| ADR-0009 | Consistencia por imagen de referencia (no modelos avanzados) | Aceptado |
