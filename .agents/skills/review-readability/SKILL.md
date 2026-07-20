---
name: review-readability
description: "Trigger: readability, legibilidad, maintainability, mantenibilidad, naming, estructura, refactor. Revisión adversarial de legibilidad, estructura, nomenclatura y mantenibilidad del código."
license: Apache-2.0
metadata:
  author: team
  version: "1.0"
---

## Activation Contract

Carga esta skill cuando se requiera revisar legibilidad: nombres, estructura de carpetas, complejidad, tipos, patrones, code organization. No cargues para cambios triviales.

## Hard Rules

1. **Cada hallazgo debe tener evidencia concreta** — archivo, línea, prueba. No opiniones.
2. Clasifica severidad: **CRITICAL** (mantenibilidad severamente afectada), **WARNING** (complejidad evitable), **SUGGESTION** (mejorable).
3. No critiques estilo sin razón técnica — solo problemas que afecten mantenibilidad real.
4. Revisa el árbol completo de `app/` (backend) o `src/` (frontend).

## Execution Steps

1. Revisa nomenclatura: nombres de variables, funciones, clases, módulos consistentes?
2. Revisa estructura: organización de archivos, separación de responsabilidades, single responsibility.
3. Revisa complejidad: funciones largas, lógica anidada, código duplicado, god-components.
4. Revisa tipos: type hints (Python) o TypeScript strict mode, tipos compartidos vs duplicados.
5. Revisa estado: estado global vs local bien separado? lógica en componentes o abstraída?
6. Revisa patrones: consistencia en el estilo, convenciones del proyecto.
7. Revisa dead code: módulos, funciones o imports no utilizados.

## Output Contract

Devuelve findings[] con `location` (archivo:línea), `severity`, `claim`, `evidence_class`, `causal_disposition`, `proof_refs[]`. Incluye evidence[] con todo lo revisado.

## References

- `.agents/reviews/` — Reportes 4R anteriores.
