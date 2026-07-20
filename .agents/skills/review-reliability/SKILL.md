---
name: review-reliability
description: "Trigger: reliability, confiabilidad, tests, errores, error handling, edge cases, regresiones, bugs. Revisión adversarial de confiabilidad, tests, manejo de errores y regresiones."
license: Apache-2.0
metadata:
  author: team
  version: "1.0"
---

## Activation Contract

Carga esta skill cuando se requiera revisar confiabilidad: tests, manejo de errores, edge cases, estados de carga/vacío/error, regresiones. No cargues para cambios triviales.

## Hard Rules

1. **Cada hallazgo debe tener evidencia concreta** — archivo, línea, prueba. No opiniones.
2. Clasifica severidad: **CRITICAL** (bug en producción seguro), **WARNING** (comportamiento frágil), **SUGGESTION** (mejorable).
3. Revisa el árbol completo de `app/` (backend) y `tests/`, o `src/` (frontend).

## Execution Steps

1. Revisa tests: existen? cubren casos borde? son deterministas? se pueden ejecutar?
2. Revisa manejo de errores: try/catch, status codes HTTP, respuestas consistentes.
3. Revisa edge cases: qué pasa cuando una API externa falla, DB caída, token expirado.
4. Revisa estados de UI (frontend): loading, empty, error states en todos los componentes.
5. Revisa TanStack Query (frontend): query keys, stale times, refetch, mutations.
6. Revisa validación de schemas (backend): Pydantic validation, constraints en DB.
7. Revisa side effects: useEffect sin memory leaks, subscriptions limpiadas.
8. Revisa background tasks (backend): manejo de fallos en operaciones asíncronas.

## Output Contract

Devuelve findings[] con `location` (archivo:línea), `severity`, `claim`, `evidence_class`, `causal_disposition`, `proof_refs[]`. Incluye evidence[] con todo lo revisado.

## References

- `.agents/reviews/` — Reportes 4R anteriores.
