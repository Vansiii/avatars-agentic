# Feature Spec: Creación de Personajes

> **ID:** FEAT-001  
> **Versión:** 2.0  
> **Estado:** Planificado  
> **Prioridad:** P0 — Crítico  
> **Fase:** 002

---

## Descripción

Flujo mediante el cual un usuario crea un personaje hiperrealista para spots de TV, subiendo una imagen de referencia, escribiendo una descripción, o ambos.

---

## Criterios de Aceptación

### AC-001: Entrada de datos
- [ ] El sistema acepta imagen JPEG/PNG/WEBP (≤ 10 MB)
- [ ] El sistema acepta prompt de texto (10-500 caracteres)
- [ ] Se puede enviar solo imagen, solo texto, o ambos
- [ ] Al menos uno de los dos es obligatorio

### AC-002: Validación de imagen
- [ ] Se valida formato (GEN_001 si no soportado)
- [ ] Se valida tamaño (GEN_002 si > 10 MB)
- [ ] Se valida que la imagen no sea basura (contenido no recognizable)

### AC-003: Generación
- [ ] El sistema genera 3 variaciones de imagen hiperrealista
- [ ] Cada variación pasa por filtro NSFW
- [ ] Las variaciones se muestran al usuario para selección

### AC-004: Selección
- [ ] El usuario puede elegir una de las 3 variaciones
- [ ] Al elegir, se guarda como personaje activo
- [ ] Se guarda la imagen de referencia para consistencia futura

### AC-005: Rehacer
- [ ] El usuario puede rehacer hasta 3 veces gratis
- [ ] Cada rehacer genera 3 variaciones nuevas
- [ ] No se decrementa el límite hasta selección final

### AC-006: Límites
- [ ] Máximo 2 personajes por semana por usuario
- [ ] Al alcanzar límite: 403 con código CHAR_001
- [ ] Límites se resetean cada lunes 00:00 UTC

---

## Flujo de Usuario

```
1. Usuario navega a /app/characters/new
2. Sube imagen y/o escribe descripción
3. Hace clic en "Crear Personaje"
4. Sistema genera 3 variaciones (muestra progreso)
5. Usuario ve las 3 opciones
6. Si le gusta una → la elige → personaje creado
7. Si no le gusta → "Rehacer" (máx 3 veces)
8. Después de 3 rehacers → DEBE elegir una
```

---

## Especificaciones Técnicas

### Endpoint

```
POST /api/v1/characters
Content-Type: multipart/form-data

Fields:
  - name: string (requerido)
  - description: string (opcional)
  - prompt: string (opcional, 10-500 chars)
  - file: file (opcional, JPEG/PNG/WEBP ≤ 10MB)
  - category: string (requerido)

Response 202:
{
  "status": "pending",
  "data": {
    "character_id": "uuid",
    "estimated_seconds": 30
  }
}
```

### Selección

```
POST /api/v1/characters/{id}/select
Body: { "variation_index": 0 }

Response 200:
{
  "status": "success",
  "data": {
    "character_id": "uuid",
    "selected": true,
    "remaining_characters_this_week": 1
  }
}
```

### Rehacer

```
POST /api/v1/characters/{id}/redo

Response 202:
{
  "status": "pending",
  "data": {
    "character_id": "uuid",
    "redos_remaining": 2,
    "estimated_seconds": 30
  }
}
```

---

## Casos de Error

| Código | HTTP | Significado |
|---|---|---|
| CHAR_001 | 403 | Límite semanal de personajes alcanzado |
| GEN_001 | 400 | Formato de imagen no soportado |
| GEN_002 | 400 | Imagen > 10 MB |
| GEN_004 | 422 | Contenido rechazado por NSFW |
