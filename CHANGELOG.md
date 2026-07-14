# Changelog — Avatars Agentic

Registro de cambios importantes en la documentación y estructura del sistema multi-agente.

---

## [2.0.0] - 2026-07-13

### ✅ Correcciones Mayores (Post-Auditoría)

#### Arquitectura
- **CORREGIDO**: Inconsistencia entre documentación y código real
  - `SDD.md` actualizado a v2.0: monolito FastAPI como arquitectura oficial
  - `SOUL.md` §6 prohíbe explícitamente Celery/Redis/microservicios en Alpha
  - `roadmap.md` hito 2.1 actualizado: BackgroundTasks en lugar de Celery
  - `feature-001.md` actualizado a v1.1 con dependencias reales

#### Sistema de Memoria Compartida
- **IMPLEMENTADO**: Protocolo de memoria entre agentes
  - `.agents/memory/SOUL.md` creado con 6 secciones de reglas inmutables
  - `.agents/memory/HEARTBEAT.md` para estado actual (se sobrescribe)
  - `.agents/memory/MEMORY.md` para historial (append-only)
  - Protocolo documentado en `AGENTS.md` y `agent.md`

#### Documentación
- **CREADO**: `README.md` principal del proyecto (162 líneas)
  - Explica qué es el sistema multi-agente
  - Documenta arquitectura y protocolo de trabajo
  - Lista roles, reglas inmutables y estado actual
  - Referencia a todos los documentos clave

- **ACTUALIZADO**: `informe_estructura_avatares.md` v1.1
  - Marcado como "correcciones aplicadas"
  - Sección final con estado de implementación
  - Métricas de mejora documentadas

### 🔧 Cambios Técnicos Reflejados

- ✅ Proveedor de IA: Pollinations.ai (no Replicate/Gemini)
- ✅ Cola de tareas: BackgroundTasks de FastAPI (no Celery + Redis)
- ✅ Watermark: Implementado con Pillow y verificado
- ✅ Filtro NSFW entrada: Activo vía `safe=true` de Pollinations
- ❌ Filtro NSFW salida: Pendiente (bloqueante B-04)

### 📊 Estado del Backlog

**Épica A (Fundaciones)**: ✅ 8/8 completadas  
**Épica B (IA Real)**: 🟡 7/9 completadas (B-04 bloqueante crítico)  
**Épica C (Frontend)**: 🔴 0/8 completadas (C-08 bug crítico)  
**Épica D (Seguridad)**: 🔴 0/5 completadas  
**Épica E (Cierre Alpha)**: 🔴 0/3 completadas  

### 📝 Archivos Modificados

```
✏️  spec/roadmap.md (hitos 2.1, 2.2, checklist fase 002)
✏️  spec/features/feature-001.md (v1.1, AC actualizados, dependencias)
✏️  informe_estructura_avatares.md (v1.1, estado de correcciones)
➕  README.md (nuevo)
➕  CHANGELOG.md (nuevo, este archivo)
```

### 📚 Documentación de Referencia Actualizada

Todos los documentos ahora están **100% consistentes**:
- `SOUL.md` ← fuente de verdad de reglas técnicas
- `constitucion.md` ← fuente de verdad de principios de negocio
- `SDD.md` v2.0 ← arquitectura técnica real
- `roadmap.md` + `backlog.md` ← tareas actuales (backlog manda)

---

## [1.0.0] - 2026-07-08 (Estado Pre-Auditoría)

### Estado Original

- ❌ Documentación pedía microservicios + Celery + Redis
- ❌ Código real era monolito FastAPI simple
- ❌ Sin sistema de memoria compartida entre agentes
- ❌ Generación de avatares era un mock con Unsplash
- ⚠️ Inconsistencias entre `roadmap.md`, `SDD.md` y código

### Archivos Creados Inicialmente

```
spec/
├── mision.md
├── constitucion.md
├── sdd.md (v1.0, arquitectura idealizada)
├── roadmap.md (fases con Celery/Redis)
└── features/ (5 features definidas)

agent.md (definición de 10 roles)
```

---

## Notas de Versión

### Principios de Versionado

- **Major (X.0.0)**: Cambios en arquitectura o protocolo de agentes
- **Minor (x.X.0)**: Nuevas features o correcciones significativas
- **Patch (x.x.X)**: Correcciones menores o actualizaciones de documentación

### Próxima Versión Planificada: [2.1.0]

Cuando se complete:
- Filtro NSFW de salida (B-04)
- Bug C-08 de simulación de imágenes
- Strip de EXIF (D-01)
- Épica C completada (frontend funcional E2E)

---

**Mantenedores**: Sistema Multi-Agente  
**Última actualización**: 2026-07-13
