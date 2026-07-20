---
name: review-resilience
description: "Trigger: resilience, resiliencia, fallos, failures, recovery, degraded, degraded mode, retry, backoff, timeouts. Revisión adversarial de tolerancia a fallos, degradación graceful y recovery."
license: Apache-2.0
metadata:
  author: team
  version: "1.0"
---

## Activation Contract

Carga esta skill cuando se requiera revisar resiliencia: dependencias externas, fallos parciales, timeouts, retry, degraded mode, recovery. No cargues para cambios triviales.

## Hard Rules

1. **Cada hallazgo debe tener evidencia concreta** — archivo, línea, prueba. No opiniones.
2. Clasifica severidad: **CRITICAL** (sistema inoperable ante fallo), **WARNING** (disponibilidad afectada), **SUGGESTION** (mejorable).
3. Revisa el árbol completo de `app/` (backend) o `src/` (frontend).

## Execution Steps

1. Revisa dependencias externas (APIs, DB): timeouts, retry/backoff configurados?
2. Revisa manejo de fallos parciales: qué pasa si una parte del request falla?
3. Revisa degraded mode: qué features siguen andando si una dependencia externa está caída?
4. Revisa conexiones DB: pool_pre_ping, pool_recycle, connection handling.
5. Revisa background tasks: manejo de fallos, reintentos, estado inconsistente.
6. Revisa startup health checks: la app falla rápido si falta configuración crítica?
7. Revisa frontend: loading states, error boundaries, timeouts en fetch, onError en media.
8. Revisa rate limiting / throttling: hay protección contra abusos?

## Output Contract

Devuelve findings[] con `location` (archivo:línea), `severity`, `claim`, `evidence_class`, `causal_disposition`, `proof_refs[]`. Incluye evidence[] con todo lo revisado.

## References

- `.agents/reviews/` — Reportes 4R anteriores.
