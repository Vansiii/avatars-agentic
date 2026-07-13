# Feature Spec: Generación de Avatares con IA

> **ID:** FEAT-001  
> **Versión:** 1.0  
> **Estado:** En desarrollo  
> **Prioridad:** P0 — Crítico (Core MVP)

---

## Descripción

Flujo completo mediante el cual un usuario sube una fotografía o escribe una descripción textual, selecciona un estilo visual y recibe hasta 6 variaciones de avatar generadas por inteligencia artificial.

---

## Criterios de Aceptación

### AC-001: Ingesta de imagen
- [ ] El sistema acepta imágenes en formato JPEG, PNG y WEBP.
- [ ] El tamaño máximo permitido es 10 MB.
- [ ] Si la imagen supera el límite, el sistema muestra un error claro antes de intentar el upload.
- [ ] La resolución mínima aceptada es 200 × 200 px.

### AC-002: Ingesta de descripción textual
- [ ] El usuario puede escribir una descripción libre en lugar de subir foto.
- [ ] La descripción tiene un mínimo de 10 caracteres y máximo de 500.
- [ ] El campo muestra un contador de caracteres en tiempo real.

### AC-003: Selección de estilo
- [ ] El usuario ve una galería visual de estilos disponibles con previews.
- [ ] Los estilos bloqueados (tier superior) muestran un candado con indicación del plan requerido.
- [ ] El usuario puede seleccionar exactamente 1 estilo por generación.

### AC-004: Validación de cuota
- [ ] Antes de encolar el job, el sistema verifica que el usuario tiene créditos disponibles.
- [ ] Si no hay créditos, se muestra un CTA para actualizar el plan.
- [ ] El crédito se descuenta **al completar** la generación exitosamente, no al encolar.

### AC-005: Feedback durante la generación
- [ ] Al encolar el job, el usuario ve una pantalla de espera con progreso estimado.
- [ ] El progreso se actualiza vía WebSocket (sin necesidad de refrescar la página).
- [ ] Si la generación tarda más de 120 segundos, se muestra un error y no se descuenta crédito.

### AC-006: Presentación de resultados
- [ ] Se presentan entre 3 y 6 variaciones del avatar generado.
- [ ] Cada variación muestra una versión reducida (preview 256 × 256).
- [ ] El usuario puede hacer clic en una variación para verla en resolución completa.

### AC-007: Descarga
- [ ] El usuario free puede descargar en resolución 512 × 512 px (PNG).
- [ ] El usuario pro/enterprise puede descargar en resolución 1024 × 1024 px (PNG).
- [ ] La descarga premium (2048 × 2048) requiere plan enterprise o pago individual.
- [ ] Los avatares free incluyen watermark sutil en la esquina inferior derecha.

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

- **FEAT-003:** Catálogo de Estilos (debe estar disponible antes de implementar el selector)
- **FEAT-004:** Gestión de Usuarios y Créditos (validación de cuota)
- Servicio externo: Replicate API / DALL-E API
- Infraestructura: Redis + Celery Workers desplegados

---

## Métricas de Éxito

| Métrica | Target |
|---|---|
| Tiempo promedio de generación | < 60 segundos |
| Tasa de éxito de generaciones | > 95% |
| Tasa de descarga tras ver resultados | > 70% |
| Tasa de solicitud de modificación | < 30% |
