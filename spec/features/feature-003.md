# Feature Spec: Gestión de Personajes

> **ID:** FEAT-003  
> **Versión:** 2.0  
> **Estado:** Planificado  
> **Prioridad:** P1 — Alta  
> **Fase:** 004

---

## Descripción

CRUD completo de personajes: listar, ver detalles, editar, eliminar y reasignar a categorías.

---

## Criterios de Aceptación

### AC-001: Listar personajes
- [ ] El usuario ve todos sus personajes
- [ ] Se muestra imagen, nombre, categoría y estado
- [ ] Paginación funcional

### AC-002: Ver detalle
- [ ] Al hacer clic, se ve detalle completo del personaje
- [ ] Se muestra imagen generada, descripción, categoría
- [ ] Se muestran los spots asociados

### AC-003: Editar personaje
- [ ] Se puede cambiar nombre, descripción, categoría
- [ ] Se puede regenerar la imagen (nuevas variaciones)
- [ ] Los cambios no afectan spots ya generados

### AC-004: Eliminar personaje
- [ ] Confirmación antes de eliminar
- [ ] Eliminar personaje no elimina spots ya generados
- [ ] Los spots quedan huérfanos (se pueden ver pero no regenerar)

### AC-005: Reasignar categoría
- [ ] Se puede mover un personaje a otra categoría
- [ ] El personaje ahora aparece en spots de la nueva categoría

---

## Especificaciones Técnicas

### Endpoints

```
GET    /api/v1/characters           → Lista de personajes del usuario
GET    /api/v1/characters/{id}      → Detalle de personaje
PUT    /api/v1/characters/{id}      → Actualizar personaje
DELETE /api/v1/characters/{id}      → Eliminar personaje
POST   /api/v1/characters/{id}/regenerate  → Regenerar imagen
```
