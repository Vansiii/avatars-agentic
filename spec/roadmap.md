# Hoja de Ruta de Implementación
## Sistema de Personajes para Spots Publicitarios de TV

> **Última actualización:** 2026-07-14  
> **Versión:** 2.0

---

## Visión General

```
FASE 001  ──►  FASE 002  ──►  FASE 003  ──►  FASE 004
  Base            Personaje       Video          Producción
  (Sem 1-2)       (Sem 2-4)       (Sem 4-6)      (Sem 6-8)
```

---

## Fase 001: Base del Sistema

> **Duración:** 2 semanas  
> **Estado:** Planificado

### Hitos

| # | Hito | Criterio de Done |
|---|---|---|
| 1.1 | Setup de proyecto: FastAPI + React + PostgreSQL | `docker-compose up` levanta todo |
| 1.2 | Auth: login/registro de admin, JWT | Admin puede loguearse |
| 1.3 | Gestión de usuarios: admin crea usuarios | Admin crea usuario, usuario se loguea |
| 1.4 | Modelos de datos: users, characters, spots | DB crea tablas correctamente |
| 1.5 | Frontend: estructura PWA con rutas | App carga, routing funcional |

### Checklist de Cierre
- [ ] Flujo completo: admin crea usuario → usuario se loguea
- [ ] DB crea todas las tablas
- [ ] Frontend compila sin errores

---

## Fase 002: Creación de Personajes

> **Duración:** 2 semanas  
> **Prerequisito:** Fase 001  
> **Estado:** Planificado

### Hitos

| # | Hito | Criterio de Done |
|---|---|---|
| 2.1 | Endpoint POST /characters (imagen +/o texto) | Personaje creado en DB |
| 2.2 | Pipeline de generación de imagen con Pollinations | Se genera imagen hiperrealista |
| 2.3 | Filtro NSFW en entrada y salida | Contenido NSFW rechazado |
| 2.4 | Selección de variación (3 opciones) | Usuario elige variación |
| 2.5 | Sistema de rehacer (3 veces) | Rehacer funciona, no decrementa límite |
| 2.6 | Límites semanales (2 personajes) | 403 al alcanzar límite |
| 2.7 | Frontend: página de crear personaje | UI funcional con flujo completo |
| 2.8 | Consistencia: guardar reference_image_url | Personaje tiene imagen de referencia |

### Checklist de Cierre
- [ ] Flujo completo: crear → generar 3 variaciones → elegir → personaje guardado
- [ ] Rehacer funciona (3 veces gratis)
- [ ] Límite semanal se respeta
- [ ] NSFW filtra correctamente

---

## Fase 003: Generación de Spots (Video)

> **Duración:** 2 semanas  
> **Prerequisito:** Fase 002  
> **Estado:** Planificado

### Hitos

| # | Hito | Criterio de Done |
|---|---|---|
| 3.1 | Endpoint POST /spots (personaje + script + tipo) | Spot creado en DB |
| 3.2 | Pipeline de generación de video | Se genera video del personaje |
| 3.3 | Consistencia: personaje aparece en el video | El personaje se reconoce |
| 3.4 | Tipos de video: short (3-5s) y long (15-30s) | Ambos tipos funcionan |
| 3.5 | Selección de variación de video | Usuario elige variación |
| 3.6 | Sistema de rehacer para spots | Rehacer funciona |
| 3.7 | Límites semanales (5 spots) | 403 al alcanzar límite |
| 3.8 | Frontend: página de generar spot | UI funcional |

### Checklist de Cierre
- [ ] Flujo completo: seleccionar personaje → describir spot → generar → elegir
- [ ] Consistencia del personaje verificada
- [ ] Ambos tipos de video funcionan
- [ ] Límite semanal se respeta

---

## Fase 004: Cierre del Alpha

> **Duración:** 2 semanas  
> **Prerequisito:** Fase 003  
> **Estado:** Planificado

### Hitos

| # | Hito | Criterio de Done |
|---|---|---|
| 4.1 | Gestión de personajes: editar, eliminar, reasignar | CRUD completo funcional |
| 4.2 | Categorías de spots (deportes, noticias, etc.) | Categorías funcionan |
| 4.3 | Dashboard con métricas de uso | Admin ve métricas |
| 4.4 | Tests E2E | Flujo completo testeado |
| 4.5 | README de arranque | Cómo levantar el sistema |

### Checklist de Cierre
- [ ] CRUD de personajes completo
- [ ] Categorías funcionan
- [ ] Tests pasan
- [ ] README actualizado

---

## Features

| ID | Feature | Fase | Estado |
|---|---|---|---|
| FEAT-001 | Creación de personajes | 002 | Planificado |
| FEAT-002 | Generación de spots | 003 | Planificado |
| FEAT-003 | Gestión de personajes | 004 | Planificado |
| FEAT-004 | Gestión de usuarios | 001 | Planificado |
| FEAT-005 | Categorías de spots | 004 | Planificado |
