---
name: review-risk
description: "Trigger: risk, riesgo, security, seguridad, data exposure, exposición de datos, architecture review. Revisión adversarial de seguridad, permisos, exposición de datos, arquitectura y dependencias."
license: Apache-2.0
metadata:
  author: team
  version: "1.0"
---

## Activation Contract

Carga esta skill cuando se requiera revisar riesgos de seguridad, permisos, exposición de datos, arquitectura o dependencias. No cargues para cambios triviales (typos, docs).

## Hard Rules

1. **Cada hallazgo debe tener evidencia concreta** — archivo, línea, prueba. No opiniones.
2. Clasifica severidad: **CRITICAL** (vulnerabilidad explotable), **WARNING** (riesgo importante), **SUGGESTION** (mejorable).
3. No ignores hallazgos preexistentes — documéntalos con `causal_disposition: pre-existing`.
4. Revisa el árbol completo de `app/` (backend) o `src/` (frontend).

## Execution Steps

1. Escanea secrets/defaults hardcodeados (SECRET_KEY, API keys, passwords).
2. Revisa autenticación: JWT hardening, token storage, refresh rotation, rate limiting.
3. Revisa autorización: role checks, endpoint protection, missing guards.
4. Revisa exposición de datos: PII en responses, tokens en logs, error messages.
5. Revisa dependencias: librerías con vulnerabilidades conocidas, versiones.
6. Revisa configuración: CORS, CSP, security headers, Dockerfile.
7. Revisa validación de inputs: SQL injection, XSS, schema validation, file uploads.

## Output Contract

Devuelve findings[] con `location` (archivo:línea), `severity`, `claim`, `evidence_class`, `causal_disposition`, `proof_refs[]`. Incluye evidence[] con todo lo revisado.

## References

- `.agents/reviews/` — Reportes 4R anteriores.
