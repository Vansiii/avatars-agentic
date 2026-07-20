# 4R — Revisiones Adversariales de Código

Las **4R** son cuatro revisiones adversariales independientes que cubren distintos aspectos de calidad de software. Cada R es un lens específico que se aplica según la señal dominante del cambio.

---

## Las 4R

### 🛡️ R1 — Risk (Riesgo)

| Aspecto | Qué busca |
|---------|-----------|
| Seguridad | JWT hardening, secrets, CORS, CSP, security headers, rate limiting |
| Permisos | Role checks, endpoint protection, privilege escalation |
| Datos | Exposición de PII, tokens en localStorage, errores con stack traces |
| Arquitectura | Dependencias vulnerables, deuda técnica estructural |
| Inputs | SQL injection, XSS, validación de schemas, file uploads |

**Cuándo cargarla:** Cambios en auth, endpoints públicos, manejo de secrets, configuraciones de seguridad, o cualquier código que maneje datos sensibles.

**Skill:** `.agents/skills/review-risk/SKILL.md`

---

### 🔄 R2 — Readability (Legibilidad)

| Aspecto | Qué busca |
|---------|-----------|
| Nomenclatura | Nombres de variables, funciones, clases, archivos |
| Estructura | Organización del proyecto, separación de concerns |
| Complejidad | God-components, funciones largas, lógica anidada, duplicación |
| Tipos | Type hints, strict mode, tipos compartidos vs duplicados |
| Estado | Global vs local, lógica en componentes vs abstraída |

**Cuándo cargarla:** Refactors, cambios en componentes grandes, reorganización de archivos, o cuando se introduce nueva lógica de negocio.

**Skill:** `.agents/skills/review-readability/SKILL.md`

---

### ⚡ R3 — Reliability (Confiabilidad)

| Aspecto | Qué busca |
|---------|-----------|
| Tests | Cobertura, casos borde, determinismo, ejecutabilidad |
| Errores | try/catch, status codes HTTP, mensajes consistentes |
| Edge cases | APIs externas caídas, DB down, tokens expirados |
| Estados UI | Loading, empty, error en todos los componentes |
| Side effects | Memory leaks, subscriptions, useEffect sin cleanup |

**Cuándo cargarla:** Cambios en lógica de negocio crítica, endpoints que llaman APIs externas, nuevo flujo de usuario, o cuando no hay tests.

**Skill:** `.agents/skills/review-reliability/SKILL.md`

---

### 🔁 R4 — Resilience (Resiliencia)

| Aspecto | Qué busca |
|---------|-----------|
| Dependencias externas | Timeouts, retry/backoff, degraded mode |
| Fallos parciales | Qué pasa si una parte del request falla |
| Conexiones DB | Pool management, pool_pre_ping, failover |
| Background tasks | Manejo de fallos, reintentos, consistencia |
| Health checks | Startup validation, readiness probes |

**Cuándo cargarla:** Cambios que integran APIs externas, modificaciones en servicios de base de datos, background tasks, o configuraciones de deployment.

**Skill:** `.agents/skills/review-resilience/SKILL.md`

---

## ¿Cuándo aplicar cuál?

| Señal dominante | Lens |
|---|---|
| Seguridad, permisos, datos sensibles, arquitectura | **R1 — Risk** |
| Refactors, naming, organización, duplicación | **R2 — Readability** |
| Tests, errores, edge cases, estados de UI | **R3 — Reliability** |
| APIs externas, fallos, recovery, degraded mode | **R4 — Resilience** |
| Hot path (auth/security/payments) o >400 líneas | **Las 4** |

## Clasificación de hallazgos

| Severidad | Significa | Acción recomendada |
|-----------|-----------|-------------------|
| 🔴 **CRITICAL** | Vulnerabilidad explotable o bug de producción | Arreglar antes del próximo deploy |
| 🟡 **WARNING** | Riesgo importante o comportamiento frágil | Revisar y planificar fix |
| 🔵 **SUGGESTION** | Mejora de calidad o mantenibilidad | Agendar para backlog |

## Causalidad (causal_disposition)

| Disposición | Significa |
|-------------|-----------|
| `introduced` | El hallazgo fue introducido por el cambio actual |
| `pre-existing` | El hallazgo existía antes del cambio |
| `base-only` | Solo observable en la base/base-ref, no en el cambio |
| `unknown` | No se pudo determinar causalidad |

## Cómo ejecutar una revisión 4R

```bash
# Si gentle-ai está disponible:
gentle-ai review start --cwd <repo-path>
gentle-ai review finalize --result <resultado.json>

# O manual cargando la skill correspondiente en el agente:
# 1. Cargá la skill del lens que necesitás
# 2. Revisá el árbol de código
# 3. Devolvé findings estructurados
```

## Reportes existentes

| Fecha | Reporte | Archivo |
|-------|---------|---------|
| 2026-07-20 | Backend + Frontend — 4R completo | `4R-2026-07-20-backend-frontend.md` |
