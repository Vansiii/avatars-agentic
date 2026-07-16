# Constitución del Sistema — Personajes para Spots Publicitarios de TV

> **Versión:** 2.0  
> **Fecha:** 2026-07-14  
> **Estado:** Ratificado  
> **Autoridad:** Documento fundacional — no puede modificarse sin aprobación del Tech Lead y Product Owner

---

## 1. Propósito de este Documento

Esta Constitución define los **invariantes del sistema**: principios, restricciones y contratos que no pueden violarse bajo ninguna circunstancia durante el desarrollo, operación o evolución de la plataforma.

---

## 2. Principios Fundacionales

### 2.1 Privacidad por Diseño
- Las imágenes subidas por usuarios **nunca se comparten** con terceros sin consentimiento.
- Las fotos de entrada se eliminan **dentro de las 24 horas**.
- El sistema **no entrena modelos** con las imágenes de usuarios sin consentimiento.

### 2.2 Transparencia de IA
- El sistema **siempre informa** que el contenido fue generado por inteligencia artificial.
- No se permite presentar contenido generado como real sin marcado explícito.

### 2.3 Contenido Seguro (Safety First)
- El sistema **rechaza activamente** contenido que vulnere los derechos de menores.
- Se aplican filtros NSFW en todas las generaciones, sin excepción.
- Las violaciones se registran en un log inmutable de seguridad.

### 2.4 Consistencia del Personaje
- Un personaje creado se mantiene **visualmente consistente** en todos los spots de su categoría.
- La imagen de referencia del personaje se usa en cada generación de video.
- Solo el usuario puede modificar o reasignar un personaje.

### 2.5 Portabilidad del Usuario
- El usuario puede **exportar y eliminar** todos sus datos en cualquier momento.
- Los personajes y spots generados son **propiedad del usuario**.

---

## 3. Contratos de API

### 3.1 Contrato de Versiones
- La API sigue **versionado semántico** (`/api/v1/`).
- Los endpoints de generación deben retornar 202 Accepted en menos de **2 segundos**.

### 3.2 Formatos de Respuesta
- Todas las respuestas usan **JSON** con estructura estándar:
  ```json
  {
    "status": "success | error | pending",
    "data": { ... },
    "error": { "code": "...", "message": "..." }
  }
  ```

---

## 4. Restricciones Técnicas Inmutables

| Restricción | Valor | Justificación |
|---|---|---|
| Tamaño máximo de imagen de entrada | 10 MB | Limitación de costo/tiempo |
| Formatos de entrada | JPEG, PNG, WEBP | Soporte universal |
| Resolución de personaje | ≥ 512 × 512 px | Calidad mínima para video |
| Duración video corto | 3-5 segundos | Spots publicitarios rápidos |
| Duración video largo | 15-30 segundos | Spots completos |
| Retención de imágenes de entrada | 24 horas | Privacidad |
| Rehacer máximo | 3 veces por generación | Balance costo-calidad |
| Límite personajes/semana (free) | 2 | Control de uso |
| Límite spots/semana (free) | 5 | Control de uso |

---

## 5. Modelo de Roles y Permisos

```
ADMIN (administrador del sistema)
  ├── Crear/eliminar usuarios
  ├── Ver métricas del sistema
  └── Gestión global

USER (usuario final, creado por admin)
  ├── Crear/editar personajes (2/semana)
  ├── Generar spots (5/semana)
  ├── Rehacer generaciones (3 veces gratis)
  └── Gestionar categorías
```

**Regla de oro:** Solo el admin puede crear cuentas. No hay registro público.

---

## 6. Gobierno y Toma de Decisiones

### 6.1 Architecture Decision Records (ADR)
- Toda decisión técnica mayor se documenta en `spec/adr/ADR-XXXX.md`.

### 6.2 Proceso de Cambio a la Constitución
1. Propuesta escrita con justificación.
2. Revisión del equipo (mínimo 48 horas).
3. Aprobación de Tech Lead + Product Owner.
4. Versionado y commit.

---

## 7. Checklist de Cumplimiento

Antes de cada release:
- [ ] Los filtros NSFW están activos y testeados
- [ ] La consistencia del personaje funciona en spots
- [ ] Los límites semanales se respetan
- [ ] El proceso de rehacer funciona correctamente
- [ ] La retención de imágenes está configurada a 24h
