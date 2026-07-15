# Security Runbook

## Rotation status

**Credential rotation is NOT performed by this change.** This document records the required procedure only. Rotating the credentials below remains separately authorized operational work and must be performed by approved service administrators with the security owner informed.

## Credentials requiring rotation

| Credential | Owning service | Operational owner | Required approval |
| --- | --- | --- | --- |
| Supabase password | Supabase project / service account | Supabase project or service owner | Supabase administrator and security owner |
| JWT `SECRET_KEY` | Backend authentication service | Backend service owner | Backend administrator and security owner |

Do not place credential values in tickets, source control, shell history, chat transcripts, or this runbook.

## Ordered rotation procedure

1. **Pre-rotation.** Open a security incident/change record, identify all consumers, confirm administrator access, schedule a change window, notify affected owners, and prepare a secure rollback path. Capture current health/authentication baselines without recording secret material.
2. **Generate replacement credentials.** Generate independent high-entropy replacement values using an approved secret-management workflow. Store them only in the approved secret store.
3. **Update the owning service.** Update the Supabase credential at the Supabase project/service owner boundary and the JWT `SECRET_KEY` at the backend authentication-service configuration boundary. Use approved configuration deployment mechanisms; do not commit values to the repository.
4. **Update dependent consumers.** Roll out references from the approved secret store to each identified backend, worker, deployment, and integration consumer. Use a documented overlap/dual-key window only where the owning service explicitly supports it.
5. **Validate.** Verify authentication and authorization flows, consumer connectivity, service health, error rates, and audit logs. Confirm old credentials no longer authenticate once the approved transition window closes.
6. **Rollback or incident response.** If validation fails, follow the change record's rollback plan (for example, temporarily restore the previous approved secret reference only if still safe), limit access, notify security and service owners, and preserve audit evidence. Treat suspected exposure or unauthorized use as an incident.

## Post-rotation repository handling

- Run the approved repository/CI secret scanner and inspect changed configuration references to ensure rotated credentials were not reintroduced.
- Verify `.env` and other local-secret files remain excluded from version control; use example/template environment files without values for documentation.
- Record rotation completion and validation evidence in the authorized operational system, not in source control.
- Git history rewriting is out of scope: it is disruptive, does not invalidate already exposed credentials, and is not a substitute for service-side rotation. Any history-remediation decision requires separate security and repository-owner authorization.
