# Feature Spec: Categorías de Spots

> **ID:** FEAT-005  
> **Versión:** 2.0  
> **Estado:** Planificado  
> **Prioridad:** P1 — Alta  
> **Fase:** 004

---

## Descripción

Sistema de categorías para organizar spots. Cada categoría puede tener un personaje asignado que aparece en todos los spots de esa categoría.

---

## Criterios de Aceptación

### AC-001: Categorías predefinidas
- [ ] Deportes
- [ ] Noticias
- [ ] Entretenimiento
- [ ] Publicidad general

### AC-002: Asignar personaje a categoría
- [ ] El usuario puede asignar un personaje a una categoría
- [ ] Solo un personaje por categoría
- [ ] Al asignar, el personaje aparece en todos los spots nuevos de esa categoría

### AC-003: Cambiar personaje de categoría
- [ ] Se puede reemplazar el personaje de una categoría
- [ ] Los spots anteriores mantienen el personaje original
- [ ] Los spots nuevos usan el nuevo personaje

### AC-004: Listar categorías
- [ ] El usuario ve todas las categorías
- [ ] Se muestra el personaje asignado (si lo hay)
- [ ] Se muestra cantidad de spots en cada categoría

---

## Especificaciones Técnicas

### Endpoints

```
GET    /api/v1/categories                    → Listar categorías
PUT    /api/v1/categories/{id}/character     → Asignar personaje a categoría
GET    /api/v1/categories/{id}/spots         → Spots de una categoría
```

---

## Modelo de Datos

```sql
spot_categories
  id                    UUID PK
  name                  VARCHAR(100) UNIQUE NOT NULL
  description           TEXT
  assigned_character_id UUID FK → characters.id (nullable)
  created_at            TIMESTAMP
```
