# Changelog — TV Characters System

Registro de cambios importantes en la documentación y estructura del sistema multi-agente.

---

## [3.0.0] - 2026-07-14

### Pivote Completo del Producto

#### Nuevo Producto
- **CAMBIADO**: De "Sistema de Avatares" a "Sistema de Personajes para Spots de TV"
- Personajes hiperrealistas para spots publicitarios de televisión
- Consistencia de personaje en todos los spots de una categoría
- Generación de video (corto 3-5s / largo 15-30s)

#### Roles
- **CAMBIADO**: De free/pro/enterprise a admin/user
- Admin crea usuarios (no hay registro público)
- Usuario crea personajes y genera spots

#### Límites de Negocio
- **NUEVO**: 2 personajes por semana
- **NUEVO**: 5 spots por semana
- **NUEVO**: Rehacer 3 veces gratis (no consume crédito)

#### Estilo
- **CAMBIADO**: De múltiples estilos a solo hiperrealista

### Archivos Modificados

```
✏️  .agents/memory/SOUL.md (reescrito: 8 secciones)
✏️  .agents/AGENTS.md (reescrito: 8 secciones)
✏️  .agents/skills/frontend.md (actualizado)
✏️  .agents/skills/backend.md (actualizado)
✏️  .agents/skills/ai.md (actualizado)
✏️  .claude/agents/avatar-backend.md → tv-characters-backend.md
✏️  .claude/agents/avatar-frontend.md → tv-characters-frontend.md
✏️  .claude/agents/avatar-ai-pipeline.md → tv-characters-pipeline.md
✏️  spec/mision.md (reescrito)
✏️  spec/constitucion.md (reescrito)
✏️  spec/sdd.md (reescrito)
✏️  spec/roadmap.md (reescrito: 4 fases)
✏️  spec/features/feature-001.md (reemplazado: creación de personajes)
✏️  spec/features/feature-002.md (reemplazado: generación de spots)
✏️  spec/features/feature-003.md (reemplazado: gestión de personajes)
✏️  spec/features/feature-004.md (reemplazado: gestión de usuarios)
✏️  spec/features/feature-005.md (reemplazado: categorías de spots)
✏️  .agents/memory/HEARTBEAT.md (reemplazado)
✏️  .agents/memory/MEMORY.md (reemplazado)
✏️  .agents/steering/backlog.md (reemplazado)
✏️  README.md (reescrito)
✏️  CHANGELOG.md (este archivo)
```

### Próximos Pasos

1. Definir proveedor de video para testing
2. Crear estructura de base de datos nueva
3. Implementar Fase 001 (base del sistema)

---

## [2.0.0] - 2026-07-13

### Correcciones Mayores (Post-Auditoría)
- Sobreingeniería arquitectónica corregida
- Sistema de memoria compartida implementado
- Proveedor de IA: Pollinations.ai (no Replicate)
- BackgroundTasks (no Celery/Redis)

---

## [1.0.0] - 2026-07-08

### Estado Original
- Documentación con microservicios + Celery + Redis
- Código real era monolito FastAPI simple
- Sin memoria compartida entre agentes
- Generación de avatares era un mock

---

**Mantenedores**: Sistema Multi-Agente  
**Última actualización**: 2026-07-14
