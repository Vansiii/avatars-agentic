# Índice del Repositorio de Especificaciones

> **Proyecto:** Sistema de Creación de Identidades Visuales Digitales  
> **Versión:** 1.0  
> **Última actualización:** 2026-07-08

---

## Estructura del Repositorio

```
spec/
├── README.md                    ← Este archivo (índice general)
├── mision.md                    ← Misión, MVP, Público Objetivo, Arquitectura de Datos
├── constitucion.md              ← Principios, invariantes y gobierno del sistema
├── sdd.md                       ← Software Design Document (arquitectura técnica completa)
└── features/
    ├── avatar-generation.md     ← FEAT-001: Generación de Avatares con IA
    ├── subscriptions.md         ← FEAT-002: Sistema de Suscripciones y Créditos
    ├── style-catalog.md         ← FEAT-003: Catálogo de Estilos de Avatar
    ├── user-management.md       ← FEAT-004: Gestión de Usuarios y Perfiles
    └── corporate-avatars.md     ← FEAT-005: Avatares Corporativos para Empresas
```

---

## Documentos Fundacionales

| Documento | Descripción | Estado |
|---|---|---|
| [Misión](./mision.md) | Define el MVP, público objetivo, exclusiones y arquitectura de datos | ✅ Aprobado |
| [Constitución](./constitucion.md) | Principios, restricciones y contratos inmutables del sistema | ✅ Ratificado |
| [SDD](./sdd.md) | Diseño técnico completo: arquitectura, componentes, seguridad, datos | 🟡 Borrador |

---

## Registro de Features

| ID | Feature | Prioridad | Estado | Plan Mínimo |
|---|---|---|---|---|
| FEAT-001 | [Generación de Avatares con IA](./features/avatar-generation.md) | P0 — Crítico | 🔧 En desarrollo | Free |
| FEAT-002 | [Sistema de Suscripciones y Créditos](./features/subscriptions.md) | P1 — Alta | 📋 Planificado | — |
| FEAT-003 | [Catálogo de Estilos de Avatar](./features/style-catalog.md) | P0 — Crítico | 🔧 En desarrollo | Free |
| FEAT-004 | [Gestión de Usuarios y Perfiles](./features/user-management.md) | P0 — Crítico | 🔧 En desarrollo | Free |
| FEAT-005 | [Avatares Corporativos para Empresas](./features/corporate-avatars.md) | P1 — Alta | 📋 Planificado | Enterprise |

---

## Roadmap de Alto Nivel

```
FASE 1 — MVP (mes 1-3)
  ✦ FEAT-004: Autenticación y gestión de usuarios
  ✦ FEAT-003: Catálogo básico de estilos (9 estilos)
  ✦ FEAT-001: Pipeline de generación de avatares
  ✦ Descarga básica (512px, free tier con watermark)

FASE 2 — Monetización (mes 3-5)
  ✦ FEAT-002: Planes de suscripción + Stripe
  ✦ Resoluciones premium (1024px, 2048px)
  ✦ Historial persistente
  ✦ 20+ estilos adicionales

FASE 3 — B2B (mes 5-8)
  ✦ FEAT-005: Módulo corporativo y organizaciones
  ✦ Generación masiva por CSV
  ✦ API pública para integraciones
  ✦ Avatares animados (GIF/WebM) — FEAT-006

FASE 4 — Expansión (mes 8+)
  ✦ App móvil nativa (iOS + Android)
  ✦ Marketplace de estilos (estilos de la comunidad)
  ✦ Editor de avatar integrado
  ✦ Integración con plataformas (LinkedIn API, etc.)
```

---

## Convenciones

### Estados de Documentos
- ✅ **Aprobado / Ratificado** — Listo para implementar, no debe cambiar sin proceso formal
- 🟡 **Borrador** — En redacción, puede cambiar
- 🔧 **En desarrollo** — Feature en implementación activa
- 📋 **Planificado** — Definido pero no iniciado
- ⏸️ **En pausa** — Suspendido temporalmente
- ❌ **Descartado** — No se implementará

### Nomenclatura de Features
- Archivos: `kebab-case.md`
- IDs: `FEAT-XXX` (numeración correlativa)
- ADRs: `spec/adr/ADR-XXXX.md`

### Proceso de Actualización
1. Toda modificación a documentos `P0` requiere aprobación del Tech Lead.
2. Los cambios se registran con fecha y versión en el encabezado del documento.
3. Los features completados se marcan con ✅ en este índice.
