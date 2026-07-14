# Feature Spec: Generación de Avatares con IA

> **ID:** FEAT-001  
> **Versión:** 1.1  
> **Estado:** En desarrollo  
> **Prioridad:** P0 — Crítico (Core MVP)  
> **Última actualización:** 2026-07-13

> **Cambios v1.1:**
> - Actualizado para reflejar implementación real con Pollinations.ai (no Replicate)
> - BackgroundTasks de FastAPI (no Celery/Redis, según SOUL.md §6)
> - Criterios de aceptación marcados según estado real del backend/frontend
> - Dependencias actualizadas para reflejar lo ya implementado

---

## Descripción

Flujo completo mediante el cual un usuario sube una fotografía o escribe una descripción textual, selecciona un estilo visual y recibe hasta 6 variaciones de avatar generadas por inteligencia artificial.

---

## Criterios de Aceptación

### AC-001: Ingesta de imagen (Backend ✅ | Frontend ⏳)
- [x] El sistema acepta imágenes en formato JPEG, PNG y WEBP (validado en backend)
- [x] El tamaño máximo permitido es 10 MB (validado con error GEN_002)
- [ ] Si la imagen supera el límite, el sistema muestra un error claro antes de intentar el upload (falta en UI)
- [ ] La resolución mínima aceptada es 200 × 200 px (no validado aún)

### AC-002: Ingesta de descripción textual (Backend ✅ | Frontend ⏳)
- [x] El usuario puede escribir una descripción libre en lugar de subir foto (campo `prompt` implementado)
- [ ] La descripción tiene un mínimo de 10 caracteres y máximo de 500 (validación pendiente)
- [ ] El campo muestra un contador de caracteres en tiempo real (pendiente en UI)

### AC-003: Selección de estilo (Backend ✅ | Frontend ⏳)
- [x] El usuario ve una galería visual de estilos disponibles con previews (estilos seeded en DB)
- [ ] Los estilos bloqueados (tier superior) muestran un candado con indicación del plan requerido (C-04 backlog)
- [x] El usuario puede seleccionar exactamente 1 estilo por generación (validado en backend)

### AC-004: Validación de cuota (Backend ✅ | Frontend ⏳)
- [x] Antes de encolar el job, el sistema verifica que el usuario tiene créditos disponibles
- [x] Si no hay créditos, se retorna 403 con código GEN_003
- [ ] El UI muestra un CTA para actualizar el plan al agotar créditos (C-03 backlog)
- [x] El crédito se descuenta **al completar** la generación exitosamente, no al encolar ✅

### AC-005: Feedback durante la generación (Backend ✅ | Frontend ⏳)
- [x] Al encolar el job, el backend responde 202 con `websocket_channel`
- [x] El progreso se actualiza vía WebSocket con eventos `generation_progress`
- [ ] El UI muestra una pantalla de espera con progreso estimado (C-01 backlog)
- [x] Timeout configurado a 120 segundos, no descuenta crédito

### AC-006: Presentación de resultados (Backend ✅ | Frontend ⏳)
- [x] Se generan entre 3 y 6 variaciones del avatar (parámetro `variations`)
- [ ] El UI presenta las variaciones en grid (C-02 backlog)
- [ ] Cada variación muestra preview y opción de descarga (C-02 backlog)

### AC-007: Descarga (Backend ✅ | Frontend ⏳)
- [x] Resolución 512 × 512 px (PNG) implementada
- [ ] Resoluciones 1024 y 2048 px diferidas a post-Alpha
- [x] **Watermark universal en el Alpha** (todos los avatares, según SOUL.md §3) ✅ verificado visualmente
- [ ] Descarga desde UI pendiente (C-02 backlog)

---

## Flujo de Usuario

```
[Pantalla: /app/generate]
    │
    ├─ Paso 1: Elige tu entrada
    │      ├─ [Subir foto]  → file picker → validación → preview
    │      └─ [Describir]   → textarea con contador
    │
    ├─ Paso 2: Elige el estilo
    │      └─ Galería de tarjetas de estilo (scroll horizontal)
    │
    ├─ Paso 3: Opciones adicionales (opcional)
    │      ├─ Cantidad de variaciones (3, 4, 6)
    │      └─ Notas adicionales (texto libre, 200 chars max)
    │
    ├─ [Botón: Generar Avatar]
    │      └─ Loading screen con WebSocket progress
    │
    └─ Paso 4: Resultados
           ├─ Grid de variaciones
           ├─ [Descargar] por variación
           ├─ [Solicitar modificación] → abre modal con instrucciones
           └─ [Guardar en historial]
```

---

## Especificaciones Técnicas

### Endpoint de Creación de Job
```
POST /api/v1/generations
Content-Type: multipart/form-data

Body:
  file?         File (imagen, opcional si hay prompt)
  prompt?       string (descripción, opcional si hay file)
  style_id      UUID
  variations    integer (default: 3, max: 6)
  notes?        string (max 200 chars)

Response 202 Accepted:
{
  "status": "pending",
  "data": {
    "request_id": "uuid",
    "estimated_seconds": 45,
    "websocket_channel": "ws://api/ws/generations/{request_id}"
  }
}
```

### Evento WebSocket de Progreso
```json
{
  "event": "generation_progress",
  "data": {
    "request_id": "uuid",
    "status": "processing",
    "progress": 65,
    "message": "Aplicando estilo visual..."
  }
}
```

### Evento WebSocket de Completado
```json
{
  "event": "generation_completed",
  "data": {
    "request_id": "uuid",
    "status": "completed",
    "avatars": [
      {
        "id": "uuid",
        "preview_url": "https://cdn.example.com/avatars/preview/xxx.jpg",
        "download_url": "https://cdn.example.com/avatars/full/xxx.png",
        "resolution": "512x512",
        "is_watermarked": true
      }
    ],
    "credits_remaining": 4
  }
}
```

---

## Casos de Error

| Código | Escenario | Mensaje al Usuario |
|---|---|---|
| `GEN_001` | Formato de imagen no soportado | "El archivo debe ser JPEG, PNG o WEBP." |
| `GEN_002` | Imagen supera 10 MB | "La imagen no puede pesar más de 10 MB." |
| `GEN_003` | Sin créditos disponibles | "Has alcanzado tu límite mensual. Actualiza tu plan." |
| `GEN_004` | Contenido NSFW detectado | "La imagen no cumple con nuestras políticas de contenido." |
| `GEN_005` | Timeout de generación (>120s) | "La generación tardó demasiado. Inténtalo de nuevo." |
| `GEN_006` | Error del modelo de IA | "Ocurrió un error en la generación. No se descontó crédito." |

---

## Dependencias

- **FEAT-003:** Catálogo de Estilos ✅ (implementado con seed en el arranque)
- **FEAT-004:** Gestión de Usuarios y Créditos ✅ (validación de cuota implementada)
- **Servicio externo:** ~~Replicate API / DALL-E API~~ → **Pollinations.ai** ✅ (integrado 2026-07-13)
- **Infraestructura:** ~~Redis + Celery Workers~~ → **BackgroundTasks de FastAPI** ✅ (monolito Alpha según SOUL.md §6)

---

## Métricas de Éxito

| Métrica | Target |
|---|---|
| Tiempo promedio de generación | < 60 segundos |
| Tasa de éxito de generaciones | > 95% |
| Tasa de descarga tras ver resultados | > 70% |
| Tasa de solicitud de modificación | < 30% |
