# TV Characters System — Sistema de Personajes para Spots de TV

> **Documentación y sistema de coordinación** para el desarrollo del Sistema de Personajes para Spots Publicitarios de Televisión mediante múltiples agentes de IA especializados.

---

## ¿Qué es este repositorio?

Este **NO es el código de la aplicación**. Es el **cerebro organizativo** que coordina a múltiples agentes de IA para construir la plataforma.

Contiene:
- **Especificaciones** del producto (misión, arquitectura, features)
- **Sistema de memoria compartida** entre agentes
- **Definición de roles** especializados (Frontend, Backend, IA)
- **Backlog y roadmap** actualizados
- **Constitución del proyecto** con reglas inmutables

**Repositorios relacionados:**
- `avatars-backend/` — Código Python/FastAPI
- `avatars-frontend/` — Código React/TypeScript

---

## Arquitectura del Sistema Multi-Agente

```
.agents/
├── AGENTS.md          → Orquestador (coordinador principal)
├── memory/            → Memoria compartida
│   ├── SOUL.md       → Reglas inmutables
│   ├── HEARTBEAT.md  → Estado actual (se sobrescribe)
│   └── MEMORY.md     → Historial de decisiones (append-only)
├── skills/           → Roles especializados
│   ├── frontend.md   → Agente Frontend
│   ├── backend.md    → Agente Backend
│   └── ai.md         → Agente IA
└── steering/
    └── backlog.md    → Tareas pendientes

spec/
├── mision.md         → Definición del MVP
├── constitucion.md   → Principios inmutables
├── sdd.md            → Arquitectura técnica
├── roadmap.md        → Fases de implementación
└── features/         → Especificaciones por funcionalidad
```

---

## Roles del Sistema

| Rol | Quién | Qué hace |
|-----|-------|----------|
| **Admin** | Administrador del sistema | Crea usuarios, gestiona el sistema |
| **User** | Usuario final | Crea personajes, genera spots |

---

## Producto

Plataforma que genera **personajes hiperrealistas** para spots publicitarios de TV. Un personaje se crea una vez y aparece de forma consistente en todos los spots de su categoría.

### Límites (Plan Gratuito)

| Recurso | Límite |
|---------|--------|
| Personajes | 2 por semana |
| Spots | 5 por semana |
| Rehacer | 3 veces gratis |

---

## Stack Tecnológico

### Backend
- Python 3.11 + FastAPI
- SQLAlchemy + PostgreSQL
- JWT auth (HS256)
- BackgroundTasks (no Celery)

### Frontend
- React 19 + TypeScript + Vite
- Zustand + React Query
- CSS global

---

## Documentos Clave

### Fundacionales
- `spec/mision.md` — Qué construimos
- `spec/constitucion.md` — Principios inmutables
- `.agents/memory/SOUL.md` — Reglas técnicas

### Arquitectura
- `spec/sdd.md` — Diseño técnico
- `.agents/skills/backend.md` — Skill del agente Backend

### Gestión
- `spec/roadmap.md` — Fases
- `.agents/steering/backlog.md` — Tareas
- `.agents/memory/HEARTBEAT.md` — Estado actual

---

## Protocolo de Trabajo

1. Agente lee SOUL + HEARTBEAT + backlog
2. Identifica siguiente tarea desbloqueada
3. Carga el skill correspondiente
4. Ejecuta la tarea
5. Verifica ejecutando (no leyendo)
6. Actualiza HEARTBEAT + MEMORY + backlog

---

**Última actualización**: 2026-07-14  
**Versión**: 2.0 (pivote a Sistema de Personajes para Spots de TV)
