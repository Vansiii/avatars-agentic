# Feature Spec: Generación de Spots (Video)

> **ID:** FEAT-002  
> **Versión:** 2.0  
> **Estado:** Planificado  
> **Prioridad:** P0 — Crítico  
> **Fase:** 003

---

## Descripción

Flujo mediante el cual un usuario genera un video de spot publicitario usando un personaje existente. El personaje mantiene su apariencia en el video.

---

## Criterios de Aceptación

### AC-001: Selección de personaje
- [ ] El usuario selecciona un personaje existente (debe estar en status `active`)
- [ ] Se muestra la imagen del personaje como preview

### AC-002: Configuración del spot
- [ ] El usuario escribe la descripción del spot (10-500 caracteres)
- [ ] El usuario selecciona tipo: `short` (3-5s) o `long` (15-30s)

### AC-003: Generación
- [ ] El sistema genera 3 variaciones de video
- [ ] El personaje aparece en cada variación
- [ ] Cada variación pasa por filtro NSFW

### AC-004: Selección
- [ ] El usuario puede elegir una de las 3 variaciones
- [ ] Al elegir, se guarda el spot como completado
- [ ] Se decrementa el límite semanal de spots

### AC-005: Rehacer
- [ ] El usuario puede rehacer hasta 3 veces gratis
- [ ] No se decrementa el límite hasta selección final

### AC-006: Límites
- [ ] Máximo 5 spots por semana por usuario
- [ ] Al alcanzar límite: 403 con código VID_001

---

## Flujo de Usuario

```
1. Usuario navega a /app/spots/new
2. Selecciona un personaje de su lista
3. Describe el spot ("Pepito presentando los resultados del partido")
4. Selecciona tipo (corto/largo)
5. Hace clic en "Generar Spot"
6. Sistema genera 3 variaciones (muestra progreso)
7. Usuario ve las 3 opciones
8. Si le gusta una → la elige → spot guardado
9. Si no le gusta → "Rehacer" (máx 3 veces)
```

---

## Especificaciones Técnicas

### Endpoint

```
POST /api/v1/spots
Content-Type: application/json

Body:
{
  "character_id": "uuid",
  "script": "string (10-500 chars)",
  "type": "short | long"
}

Response 202:
{
  "status": "pending",
  "data": {
    "spot_id": "uuid",
    "estimated_seconds": 60
  }
}
```

### Selección

```
POST /api/v1/spots/{id}/select
Body: { "variation_index": 0 }

Response 200:
{
  "status": "success",
  "data": {
    "spot_id": "uuid",
    "selected": true,
    "remaining_spots_this_week": 4
  }
}
```

### Rehacer

```
POST /api/v1/spots/{id}/redo

Response 202:
{
  "status": "pending",
  "data": {
    "spot_id": "uuid",
    "redos_remaining": 2,
    "estimated_seconds": 60
  }
}
```

---

## Casos de Error

| Código | HTTP | Significado |
|---|---|---|
| VID_001 | 403 | Límite semanal de spots alcanzado |
| GEN_004 | 422 | Contenido rechazado por NSFW |
| CHAR_002 | 404 | Personaje no encontrado |
| CHAR_003 | 400 | Personaje no está en status active |
