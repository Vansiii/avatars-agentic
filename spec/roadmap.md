# Hoja de Ruta de Implementación
## Sistema de Creación de Identidades Visuales Digitales

> **Nota:** La progresión es estrictamente secuencial; no se permite el inicio de una fase sin la validación exitosa de la anterior mediante el checklist de cierre correspondiente.

---

## Visión General del Roadmap

```
FASE 001  ──►  FASE 002  ──►  FASE 003  ──►  FASE 004  ──►  FASE 005
 Entorno         IA Core        Negocio       Persistencia    Expansión
  Base           & UI           & Pagos        & Trazabilidad   B2B
 (Mes 1)      (Mes 1-2)       (Mes 2-3)       (Mes 3-4)      (Mes 5-8)
```

---

## Fase 001: Configuración de Entorno Base e Interfaz de Usuario Mínima

> **Duración estimada:** 3–4 semanas  
> **Estado:** 🔧 En desarrollo

### Objetivo
Establecer la infraestructura técnica fundacional y una interfaz funcional básica que permita a un usuario autenticarse y navegar por la plataforma.

### Hitos

| # | Hito | Feature Asociado | Criterio de Done |
|---|---|---|---|
| 1.1 | Repositorio, CI/CD y entornos (dev/staging/prod) | — | Pipeline de GitHub Actions verde en los 3 entornos |
| 1.2 | Contenerización con Docker + Docker Compose | — | `docker-compose up` levanta todos los servicios sin error |
| 1.3 | Base de datos PostgreSQL + migraciones Alembic | [FEAT-004](./features/user-management.md) | Migraciones corren sin conflictos en entorno limpio |
| 1.4 | Servicio de autenticación (registro, login, JWT) | [FEAT-004](./features/user-management.md) | Tests de integración auth pasan al 100% |
| 1.5 | Frontend: estructura PWA (React + Vite + rutas) | — | App carga en <2s, routing funcional |
| 1.6 | Pantallas base: Landing, Login, Registro, Dashboard vacío | [FEAT-004](./features/user-management.md) | Flujo registro → login → dashboard sin errores |
| 1.7 | Gestión de perfil de usuario | [FEAT-004](./features/user-management.md) | Usuario puede editar nombre y foto de perfil |

### Checklist de Cierre de Fase 001
- [ ] Todos los servicios Docker arrancan sin intervención manual
- [ ] El flujo completo Registro → Verificación de email → Login → Dashboard funciona
- [ ] Los tests de integración de autenticación pasan al 100%
- [ ] El frontend está desplegado en el entorno de staging
- [ ] La documentación de API (`/docs`) está accesible y actualizada
- [ ] El Tech Lead ha revisado y aprobado la arquitectura base

---

## Fase 002: Integración con Servicios Externos de Procesamiento (IA / Imagen)

> **Duración estimada:** 4–5 semanas  
> **Estado:** 📋 Planificado  
> **Prerequisito:** ✅ Cierre de Fase 001

### Objetivo
Implementar el pipeline central de generación de avatares con IA y exponer al usuario la experiencia completa de subir una foto, elegir un estilo y recibir resultados.

### Hitos

| # | Hito | Feature Asociado | Criterio de Done |
|---|---|---|---|
| 2.1 | Cola de tareas asíncronas (Celery + Redis) | [FEAT-001](./features/avatar-generation.md) | Job de prueba se encola, procesa y completa en staging |
| 2.2 | Integración con proveedor de IA (Replicate API) | [FEAT-001](./features/avatar-generation.md) | Generación de imagen de prueba exitosa via API |
| 2.3 | Pre-procesado de imagen de entrada (resize, EXIF strip, filtro NSFW) | [FEAT-001](./features/avatar-generation.md) | Imágenes NSFW son rechazadas; imágenes válidas se normalizan |
| 2.4 | Post-procesado: upscaling (ESRGAN) y watermark en tier free | [FEAT-001](./features/avatar-generation.md) | Avatares free tienen watermark; pro, no |
| 2.5 | Almacenamiento de resultados en Blob Storage (S3/GCS) + CDN | [FEAT-001](./features/avatar-generation.md) | URLs de avatares accesibles via CDN en <200ms |
| 2.6 | Notificación de resultados via WebSocket | [FEAT-001](./features/avatar-generation.md) | El cliente recibe el evento `generation_completed` sin polling |
| 2.7 | Catálogo de estilos: seed de 9 estilos MVP | [FEAT-003](./features/style-catalog.md) | Galería de estilos visible y seleccionable en frontend |
| 2.8 | Flujo completo de generación en UI: upload → estilo → resultados → descarga | [FEAT-001](./features/avatar-generation.md) | E2E test de generación pasa en staging |
| 2.9 | Limpieza automática de imágenes de entrada (job cron a las 24h) | [FEAT-001](./features/avatar-generation.md) | Imágenes de entrada son eliminadas pasadas 24h |

### Checklist de Cierre de Fase 002
- [ ] El pipeline completo (upload → IA → resultados) funciona de extremo a extremo
- [ ] El filtro NSFW rechaza correctamente contenido inapropiado
- [ ] Las imágenes de entrada se eliminan en el job cron de 24 horas
- [ ] El tiempo promedio de generación es < 60 segundos
- [ ] La tasa de éxito de generaciones en staging es > 95%
- [ ] Los 9 estilos MVP están en el catálogo con sus previews
- [ ] El modelo de respaldo está configurado y probado (failover del proveedor IA)

---

## Fase 003: Lógica de Negocio y Protocolos de Gestión de Errores

> **Duración estimada:** 4–5 semanas  
> **Estado:** 📋 Planificado  
> **Prerequisito:** ✅ Cierre de Fase 002

### Objetivo
Implementar el sistema de monetización (planes, créditos, suscripciones), reforzar la gestión de errores en todos los flujos críticos y añadir los estilos premium del catálogo.

### Hitos

| # | Hito | Feature Asociado | Criterio de Done |
|---|---|---|---|
| 3.1 | Sistema de créditos por usuario (cuota, descuento, recarga mensual) | [FEAT-002](./features/subscriptions.md) | Los créditos se decrementan al completar; se recargan el día 1 del mes |
| 3.2 | Planes de suscripción: Free, Pro, Enterprise (definición en DB) | [FEAT-002](./features/subscriptions.md) | Los 3 planes están en DB con sus límites correctos |
| 3.3 | Integración con Stripe Checkout (pago de suscripción) | [FEAT-002](./features/subscriptions.md) | Un pago de prueba en modo test activa el plan en < 30s |
| 3.4 | Webhooks de Stripe (payment_succeeded, payment_failed, subscription.deleted) | [FEAT-002](./features/subscriptions.md) | Los 3 eventos de webhook actualizan el estado del plan correctamente |
| 3.5 | Portal de gestión de suscripción (Stripe Customer Portal) | [FEAT-002](./features/subscriptions.md) | Usuario puede cancelar, cambiar plan y ver facturas |
| 3.6 | Bloqueo de generación por cuota agotada + CTA de upgrade | [FEAT-002](./features/subscriptions.md) | Al llegar a 0 créditos, el botón de generar se deshabilita con CTA |
| 3.7 | Email de alerta al 20% de créditos restantes | [FEAT-002](./features/subscriptions.md) | Email recibido en prueba cuando quedan ≤ 1 crédito de 5 |
| 3.8 | Gestión de errores global: retry automático, dead-letter queue | [FEAT-001](./features/avatar-generation.md) | Jobs fallidos se reintentan 3 veces; luego van a DLQ con log |
| 3.9 | Rate limiting por usuario e IP en todos los endpoints | — | Más de 20 req/min por usuario devuelve 429 |
| 3.10 | Estilos Pro y Enterprise en catálogo (20+ estilos totales) | [FEAT-003](./features/style-catalog.md) | Los estilos bloqueados muestran overlay de upgrade |

### Checklist de Cierre de Fase 003
- [ ] Un flujo completo Free→Pro (pago Stripe en modo test) funciona de extremo a extremo
- [ ] Los webhooks de Stripe están verificados con firma correcta
- [ ] Los jobs fallidos se gestionan con retry + DLQ y no descuentan crédito
- [ ] El rate limiting está activo y retorna 429 con `Retry-After` header
- [ ] Los emails transaccionales (bienvenida, créditos bajos, factura) se envían correctamente
- [ ] El panel de billing está funcional en staging

---

## Fase 004: Capa de Persistencia y Trazabilidad Histórica

> **Duración estimada:** 3–4 semanas  
> **Estado:** 📋 Planificado  
> **Prerequisito:** ✅ Cierre de Fase 003

### Objetivo
Garantizar que el usuario tenga acceso permanente a su historial de generaciones, implementar el cumplimiento de privacidad (GDPR/LGPD) y establecer un sistema de observabilidad completo para producción.

### Hitos

| # | Hito | Feature Asociado | Criterio de Done |
|---|---|---|---|
| 4.1 | Historial de generaciones paginado con thumbnails | [FEAT-004](./features/user-management.md) | Historial carga en < 500ms con paginación de 20 items |
| 4.2 | Retención diferenciada: 30 días (free) vs. permanente (pro/enterprise) | [FEAT-004](./features/user-management.md) | Job cron elimina correctamente avatares expirados |
| 4.3 | Re-descarga de avatares desde el historial | [FEAT-004](./features/user-management.md) | URLs de descarga desde historial funcionan mientras el avatar está vigente |
| 4.4 | Exportación de datos del usuario (JSON + ZIP) | [FEAT-004](./features/user-management.md) | Exportación se genera asíncronamente y llega por email en < 10 min |
| 4.5 | Eliminación de cuenta permanente | [FEAT-004](./features/user-management.md) | Todos los datos, imágenes y avatares son eliminados y verificados |
| 4.6 | Logs estructurados (JSON) en todos los servicios | — | Logs consultables en Loki/Grafana en staging |
| 4.7 | Métricas de aplicación (Prometheus + Grafana) | — | Dashboard con: latencia P95, tasa de error, jobs en cola, uso de créditos |
| 4.8 | Alertas de monitoreo (SLA 99.5%, errores > 5%) | — | Alertas configuradas y testeadas en staging |
| 4.9 | Log inmutable de seguridad (intentos NSFW, bans) | [FEAT-001](./features/avatar-generation.md) | Eventos de seguridad no pueden ser eliminados por ningún rol |
| 4.10 | Pruebas de carga (k6): 100 usuarios concurrentes | — | El sistema mantiene < 2s de latencia P95 bajo carga |

### Checklist de Cierre de Fase 004
- [ ] El historial de generaciones es funcional y paginado
- [ ] La eliminación de cuenta borra todos los artefactos del usuario (verificado en storage)
- [ ] La exportación de datos cumple con el formato GDPR
- [ ] Los dashboards de Grafana están operativos en producción
- [ ] Las pruebas de carga con k6 pasan bajo 100 usuarios concurrentes
- [ ] El SLA de 99.5% es alcanzable según las métricas de staging
- [ ] **HITO: Primera versión de producción (v1.0) disponible para usuarios reales**

---

## Fase 005: Expansión B2B y Funcionalidades Avanzadas

> **Duración estimada:** 8–12 semanas  
> **Estado:** 📋 Planificado (Ciclo 2)  
> **Prerequisito:** ✅ Cierre de Fase 004 + v1.0 en producción estable por ≥ 2 semanas

### Objetivo
Implementar el módulo corporativo B2B, la generación masiva de avatares para empresas, la API pública y los primeros avatares animados.

### Hitos

| # | Hito | Feature Asociado | Criterio de Done |
|---|---|---|---|
| 5.1 | Módulo de Organizaciones (CRUD, miembros, roles) | [FEAT-005](./features/corporate-avatars.md) | Admin puede crear org, invitar miembros y asignar créditos |
| 5.2 | Estilo corporativo personalizado (logo, paleta, fondos) | [FEAT-005](./features/corporate-avatars.md) | El estilo corporativo se aplica correctamente en la generación |
| 5.3 | Generación masiva por importación CSV | [FEAT-005](./features/corporate-avatars.md) | CSV de 20 empleados genera todos los avatares en < 15 min |
| 5.4 | Descarga masiva en ZIP por organización | [FEAT-005](./features/corporate-avatars.md) | ZIP se genera y descarga correctamente con estructura por empleado |
| 5.5 | Panel de administración de organización | [FEAT-005](./features/corporate-avatars.md) | Dashboard con uso de créditos, miembros y generaciones recientes |
| 5.6 | API pública con autenticación por API Key | — | Documentación OpenAPI publicada; cliente de prueba funcional |
| 5.7 | 30+ estilos en catálogo incluyendo estilos exclusivos Enterprise | [FEAT-003](./features/style-catalog.md) | Nuevos estilos disponibles y testeados en generación |
| 5.8 | Avatares animados: exportación en GIF y WebM | FEAT-006 (futuro) | Generación de avatar animado de 3s en resolución 512×512 |
| 5.9 | App móvil PWA mejorada (installable, push notifications) | — | App instalable desde Chrome/Safari con notificaciones push |

### Checklist de Cierre de Fase 005
- [ ] El módulo B2B pasa los criterios de aceptación de FEAT-005 al 100%
- [ ] La API pública tiene documentación completa y ejemplos de código
- [ ] Los avatares animados pasan el filtro NSFW en post-procesado
- [ ] El tiempo de generación masiva para 50 empleados es < 20 minutos
- [ ] NPS de usuarios Enterprise en beta es > 50

---

## Especificaciones de Funcionalidad (`/spec/features`)

Cada hito del roadmap tiene su correspondiente archivo de especificación detallada. Estructura estándar de una especificación de feature:

```markdown
# Feature Spec: [Nombre del Feature]

> ID: FEAT-XXX
> Versión: X.X
> Estado: [Planificado | En desarrollo | Completado]
> Prioridad: [P0 Crítico | P1 Alta | P2 Media | P3 Baja]

## Descripción
[Qué hace este feature y por qué existe]

## Criterios de Aceptación
- [ ] AC-001: ...
- [ ] AC-002: ...

## Flujo de Usuario
[Diagrama o descripción paso a paso]

## Especificaciones Técnicas
[Endpoints, modelos de datos, eventos]

## Casos de Error
[Tabla de códigos de error y mensajes]

## Métricas de Éxito
[KPIs esperados post-lanzamiento]
```

### Índice de Features por Fase

| Fase | Feature | Archivo |
|---|---|---|
| 001 | Gestión de Usuarios y Perfiles | [features/user-management.md](./features/user-management.md) |
| 002 | Generación de Avatares con IA | [features/avatar-generation.md](./features/avatar-generation.md) |
| 002 | Catálogo de Estilos de Avatar | [features/style-catalog.md](./features/style-catalog.md) |
| 003 | Sistema de Suscripciones y Créditos | [features/subscriptions.md](./features/subscriptions.md) |
| 005 | Avatares Corporativos para Empresas | [features/corporate-avatars.md](./features/corporate-avatars.md) |

---

## Dependencias entre Fases

```
FEAT-004 (Usuarios)
    └──► FEAT-001 (Generación IA)  ──► FEAT-002 (Suscripciones)
              └──► FEAT-003 (Estilos)         └──► FEAT-005 (Corporativo)
```

> Ninguna fase puede comenzar sin que sus dependencias estén en estado **Completado** y validadas por el Tech Lead.

---

## Control de Versiones del Roadmap

| Versión | Fecha | Cambio |
|---|---|---|
| 1.0 | 2026-07-08 | Versión inicial — 5 fases definidas |
