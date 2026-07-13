# Constitución del Sistema — Sistema de Creación de Identidades Visuales Digitales

> **Versión:** 1.0  
> **Fecha:** 2026-07-08  
> **Estado:** Ratificado  
> **Autoridad:** Documento fundacional — no puede modificarse sin aprobación del Tech Lead y Product Owner

---

## 1. Propósito de este Documento

Esta Constitución define los **invariantes del sistema**: principios, restricciones y contratos que no pueden violarse bajo ninguna circunstancia durante el desarrollo, operación o evolución de la plataforma. Todo cambio de arquitectura, feature o decisión técnica debe evaluarse contra este documento antes de implementarse.

---

## 2. Principios Fundacionales

### 2.1 Privacidad por Diseño (Privacy by Default)
- Las imágenes subidas por usuarios **nunca se comparten** con terceros sin consentimiento explícito.
- Las fotos de entrada se eliminan del almacenamiento temporal **dentro de las 24 horas** posteriores a la generación.
- El sistema **no entrena modelos** con las imágenes de usuarios sin consentimiento firmado y verificable.

### 2.2 Transparencia de IA
- El sistema **siempre informa** al usuario que el avatar fue generado por inteligencia artificial.
- No se permite presentar avatares generados como fotografías reales sin marcado explícito.
- Los metadatos de generación (modelo utilizado, seed, prompt base) son accesibles para el usuario propietario.

### 2.3 Contenido Seguro (Safety First)
- El sistema **rechaza activamente** la generación de contenido que vulnere los derechos de menores de edad.
- Se aplican filtros de contenido explícito (NSFW) en todas las solicitudes, independientemente del plan del usuario.
- Las violaciones detectadas se registran en un log inmutable de seguridad.

### 2.4 Disponibilidad Mínima Garantizada
- El SLA de producción es **99.5% de uptime mensual** (máximo 3.6 horas de downtime).
- Las operaciones de generación de IA son **asíncronas** para evitar timeouts y garantizar respuesta al usuario.
- El sistema debe degradar gracefully: si el modelo principal falla, activa un modelo de respaldo.

### 2.5 Portabilidad del Usuario
- El usuario puede **exportar y eliminar** todos sus datos en cualquier momento (cumplimiento GDPR/LGPD).
- Los avatares generados son **propiedad del usuario** que los encargó, sujeto a los términos de licencia de los modelos base.

---

## 3. Contratos de API

### 3.1 Contrato de Versiones
- La API pública sigue **versionado semántico** (`/api/v1/`, `/api/v2/`).
- Una versión no puede deprecarse con menos de **90 días de aviso previo** en producción.
- Los endpoints de generación deben retornar respuesta en menos de **2 segundos** (respuesta de aceptación de trabajo, no resultado final).

### 3.2 Formatos de Respuesta
- Todas las respuestas usan **JSON** con la estructura estándar:
  ```json
  {
    "status": "success | error | pending",
    "data": { ... },
    "error": { "code": "...", "message": "..." },
    "meta": { "request_id": "...", "timestamp": "..." }
  }
  ```
- Los errores **siempre** incluyen `code` (máquina) y `message` (humano).

---

## 4. Restricciones Técnicas Inmutables

| Restricción | Valor | Justificación |
|---|---|---|
| Tamaño máximo de imagen de entrada | 10 MB | Limitación de costo/tiempo de procesamiento |
| Formatos de entrada aceptados | JPEG, PNG, WEBP | Soporte universal y eficiencia |
| Resolución mínima de salida (free) | 512 × 512 px | Usable en todas las plataformas objetivo |
| Resolución máxima de salida (premium) | 2048 × 2048 px | Balance costo-calidad |
| Tiempo máximo de generación | 120 segundos | Umbral de tolerancia del usuario |
| Retención de imágenes de entrada | 24 horas | Privacidad y costo de almacenamiento |
| Retención de avatares generados (free) | 30 días | Modelo freemium sostenible |
| Idiomas de interfaz obligatorios (MVP) | ES, EN | Mercado objetivo primario |

---

## 5. Modelo de Roles y Permisos

```
SUPERADMIN
  └── ADMIN
       ├── MODERATOR        (revisa contenido flaggeado)
       └── SUPPORT          (acceso de solo lectura a cuentas)

USER (authenticated)
  ├── FREE_TIER            (5 generaciones/mes)
  ├── PRO_TIER             (100 generaciones/mes + alta resolución)
  └── ENTERPRISE_TIER      (ilimitado + API access + soporte dedicado)

ANONYMOUS                  (demo limitado: 1 generación sin registro)
```

**Regla de oro:** Ningún rol de usuario (incluyendo SUPERADMIN) puede acceder a las imágenes de entrada de otro usuario sin orden judicial documentada.

---

## 6. Gobierno y Toma de Decisiones

### 6.1 Architecture Decision Records (ADR)
- Toda decisión técnica mayor se documenta en `spec/adr/ADR-XXXX.md`.
- Un ADR no puede revertirse sin crear un nuevo ADR que lo supercede.

### 6.2 Proceso de Cambio a la Constitución
1. Propuesta escrita con justificación técnica y de negocio.
2. Revisión del equipo de ingeniería (mínimo 48 horas).
3. Aprobación de Tech Lead + Product Owner.
4. Versionado y commit en el repositorio principal con tag.

### 6.3 Deuda Técnica
- La deuda técnica se registra en `spec/tech-debt.md`.
- No se permite acumular más de **20 items de deuda técnica crítica** sin sprint de resolución.

---

## 7. Checklist de Cumplimiento

Antes de cada release a producción, verificar:

- [ ] Los filtros de contenido (NSFW) están activos y testeados
- [ ] La retención de imágenes de entrada está configurada a 24 horas
- [ ] El versioning de API es correcto y no rompe clientes existentes
- [ ] El SLA de 99.5% ha sido evaluado en staging
- [ ] El proceso de exportación/eliminación de datos de usuario funciona
- [ ] Los logs de seguridad son inmutables y están accesibles
- [ ] La documentación de API está actualizada
