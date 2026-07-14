# 🤖 Avatars Agentic — Sistema Multi-Agente de Desarrollo

> **Documentación y sistema de coordinación** para el desarrollo colaborativo del Sistema de Creación de Identidades Visuales Digitales mediante múltiples agentes de IA especializados.

---

## 📋 ¿Qué es este repositorio?

Este **NO es el código de la aplicación**. Es el **cerebro organizativo** que coordina a múltiples agentes de IA para construir la plataforma de generación de avatares.

Contiene:
- 📚 **Especificaciones completas** del producto (misión, arquitectura, features)
- 🧠 **Sistema de memoria compartida** entre agentes
- 🎭 **Definición de roles** especializados (Frontend, Backend, IA, etc.)
- 📊 **Backlog y roadmap** actualizados
- 🔒 **Constitución del proyecto** con reglas inmutables

**Repositorios relacionados:**
- `avatars-backend/` — Código Python/FastAPI (monolito)
- `avatars-frontend/` — Código React/TypeScript (PWA)

---

## 🏗️ Arquitectura del Sistema Multi-Agente

```
.agents/
├── AGENTS.md          → Orquestador (coordinador principal)
├── memory/            → Memoria compartida entre todos los agentes
│   ├── SOUL.md       → Reglas inmutables (la "constitución" técnica)
│   ├── HEARTBEAT.md  → Estado actual del proyecto (se sobrescribe)
│   └── MEMORY.md     → Historial de decisiones (append-only)
├── skills/           → Roles especializados de cada agente
│   ├── frontend.md   → Agente Frontend (React 19 + Vite)
│   ├── backend.md    → Agente Backend (FastAPI + PostgreSQL)
│   ├── ai.md         → Agente IA (Pipeline Pollinations.ai)
│   └── ...
└── steering/
    └── backlog.md    → Tareas pendientes priorizadas

spec/
├── mision.md         → Definición del MVP y público objetivo
├── constitucion.md   → Principios inmutables del sistema
├── sdd.md            → Software Design Document (arquitectura técnica)
├── roadmap.md        → Fases de implementación
└── features/         → Especificaciones detalladas por funcionalidad
    ├── feature-001.md → Generación de Avatares con IA
    ├── feature-002.md → Sistema de Suscripciones y Créditos
    ├── feature-003.md → Catálogo de Estilos
    ├── feature-004.md → Gestión de Usuarios
    └── feature-005.md → Avatares Corporativos (B2B)
```

---

## 🎯 Protocolo de Trabajo Multi-Agente

### 1️⃣ Al iniciar cualquier tarea

Todo agente **DEBE leer**:
1. `.agents/memory/SOUL.md` — Reglas que NUNCA se pueden violar
2. `.agents/memory/HEARTBEAT.md` — Estado actual del proyecto
3. `.agents/steering/backlog.md` — Qué toca hacer ahora
4. `spec/constitucion.md` — Principios del sistema
5. El archivo de la feature activa en `spec/features/`

### 2️⃣ Durante la ejecución

- Seguir estrictamente el rol definido en `.agents/skills/{rol}.md`
- No cruzar fronteras de dominio sin coordinación del Orquestador
- Verificar el código ejecutándolo, no solo leyéndolo

### 3️⃣ Al terminar una tarea

Todo agente **DEBE actualizar**:
1. `.agents/memory/HEARTBEAT.md` — Sobrescribir con el nuevo estado
2. `.agents/memory/MEMORY.md` — Añadir entrada al final (append-only)
3. `.agents/steering/backlog.md` — Marcar `[ ]` → `[x]`

---

## 📜 Reglas Inmutables (SOUL.md)

Estas reglas **NO SON NEGOCIABLES** para el Alpha:

### 🔒 Arquitectura
- ✅ **Monolito FastAPI** con BackgroundTasks
- ❌ **PROHIBIDO**: Celery, Redis, microservicios, Kubernetes, PgBouncer

### 🖼️ Generación de Avatares
- ✅ **Watermark universal** en todos los avatares del Alpha
- ✅ **Filtro NSFW** obligatorio en entrada Y salida (doble checkpoint)
- ✅ **Créditos** solo se cobran cuando la generación se COMPLETA

### 🔐 Privacidad
- ✅ Imágenes de entrada se **borran a las 24 horas**
- ✅ Strip de metadatos EXIF en toda imagen subida
- ✅ Nunca entrenar modelos con imágenes de usuarios sin consentimiento

### 💰 Planes (Alpha)
| Plan | Créditos/mes | Watermark | Resolución |
|------|--------------|-----------|------------|
| free | 5 | SÍ (siempre en Alpha) | 512×512 |
| pro | 100 | SÍ (en Alpha) | 1024×1024 |
| enterprise | ilimitado | SÍ (en Alpha) | 1024×1024 |

---

## 🚀 Stack Tecnológico Real

### Backend (monolito)
- Python 3.11 + FastAPI
- SQLAlchemy 2.0 (síncrono) + PostgreSQL 15
- BackgroundTasks (no Celery)
- JWT con HS256 (no RS256)
- **Pollinations.ai** para generación de imágenes (gratis)

### Frontend (PWA)
- React 19 + TypeScript + Vite
- Zustand + React Query
- CSS global (no CSS Modules)

---

## 📊 Estado Actual del Proyecto

**Última actualización**: 2026-07-13 (ver `HEARTBEAT.md`)

### ✅ Completado
- Estructura `.agents/` con memoria compartida
- Backend: Auth JWT, catálogo de estilos, endpoint de generación
- Generación real con Pollinations.ai (no mock)
- Watermark real con Pillow verificado visualmente
- Filtro NSFW de entrada activo (safe=true)
- WebSocket de progreso funcional

### 🚧 En progreso
- Épica B: IA Real (7/9 tareas completadas)
- Épica C: Frontend (0/8 tareas completadas)

### 🔴 Bloqueantes críticos
- **B-04**: Filtro NSFW de salida (obligatorio por Constitución §2.3)
- **C-08**: Bug que muestra imágenes falsas como reales
- **D-01**: Strip de metadatos EXIF

Ver backlog completo en `.agents/steering/backlog.md`

---

## 🎭 Roles de los Agentes

| Agente | Responsabilidad | Archivo de Skill |
|--------|----------------|------------------|
| **Orquestador** | Coordina, delega, actualiza memoria | `.agents/AGENTS.md` |
| **Frontend** | React, UI, estado del cliente | `.agents/skills/frontend.md` |
| **Backend** | FastAPI, API REST, lógica de negocio | `.agents/skills/backend.md` |
| **IA** | Pipeline de generación con IA | `.agents/skills/ai.md` |
| **Database** | Esquema, migraciones, queries | `agent.md` §database |
| **Security** | Auth, NSFW, privacidad | `agent.md` §security |
| **Testing** | Tests unitarios, integración, E2E | `agent.md` §testing |
| **DevOps** | Docker, CI/CD, despliegue | `agent.md` §devops |
| **Documentation** | Mantiene docs actualizadas, ADRs | `agent.md` §documentation |
| **Reviewer** | Audita antes de cerrar features | `agent.md` §reviewer |

---

## 📖 Documentos Clave

### Fundacionales (leer primero)
- `spec/mision.md` — ¿Qué estamos construyendo y para quién?
- `spec/constitucion.md` — Principios que NUNCA se violan
- `.agents/memory/SOUL.md` — Reglas técnicas inmutables del Alpha

### Arquitectura
- `spec/sdd.md` — Software Design Document completo
- `agent.md` — Definición detallada de todos los roles

### Gestión
- `spec/roadmap.md` — Fases de implementación
- `.agents/steering/backlog.md` — Tareas pendientes actuales
- `.agents/memory/HEARTBEAT.md` — Estado actual del proyecto
- `.agents/memory/MEMORY.md` — Historial de decisiones

---

## 🔄 Flujo de Trabajo Típico

```
1. Orquestador lee SOUL + HEARTBEAT + backlog
2. Orquestador identifica la siguiente tarea desbloqueada
3. Orquestador asigna el rol adecuado (ej: "Agente Backend")
4. El agente carga su skill (.agents/skills/backend.md)
5. El agente ejecuta la tarea respetando SOUL.md
6. El agente verifica ejecutando el código
7. El agente actualiza HEARTBEAT + MEMORY + backlog
8. Orquestador valida y coordina siguiente tarea
```

---

## 🐛 Problemas Conocidos y Correcciones Aplicadas

### ✅ Corrección: Sobreingeniería Arquitectónica
**Problema detectado** (`informe_estructura_avatares.md`):
- La documentación inicial pedía microservicios, Celery, Redis, Kubernetes
- El código real es un monolito FastAPI simple

**Solución aplicada**:
- `SOUL.md` §6 prohíbe explícitamente esa infraestructura en Alpha
- `SDD.md` v2.0 reescrito para reflejar el monolito real
- `roadmap.md` y `feature-001.md` actualizados (2026-07-13)
- Backlog es ahora la fuente de verdad, manda sobre roadmap

### ✅ Corrección: Falta de Memoria Compartida
**Problema detectado**: Agentes perdían contexto entre turnos

**Solución aplicada**:
- Sistema de memoria compartida implementado (SOUL/HEARTBEAT/MEMORY)
- Protocolo de lectura/escritura documentado y seguido
- Evidencia de uso real en `MEMORY.md`

---

## 📝 Convenciones

### Nomenclatura
- **Archivos de spec**: `kebab-case.md`
- **IDs de features**: `FEAT-XXX` (ej: FEAT-001)
- **Tareas de backlog**: `X-NN` (ej: B-04, C-08)
- **Python**: `snake_case`
- **TypeScript**: `camelCase` / `PascalCase`

### Commits y Branches
- Branches: `feature/FEAT-XXX-descripcion`
- Commits: `[FEAT-XXX] Descripción clara del cambio`

### Documentación
- Todo cambio arquitectónico requiere ADR en `spec/adr/`
- Decisiones no triviales se registran en `MEMORY.md`
- Estado actual siempre en `HEARTBEAT.md`

---

## 📞 Contacto y Soporte

Este es un proyecto de desarrollo coordinado por agentes de IA.

Para cuestiones técnicas, consulta:
- `agent.md` — Definición completa del sistema multi-agente
- `.agents/AGENTS.md` — Protocolo del Orquestador
- `spec/constitucion.md` — Principios del sistema

---

## 📜 Licencia

[Definir licencia del proyecto]

---

**Última actualización**: 2026-07-13  
**Versión de documentación**: 2.0 (post-corrección de sobreingeniería)
