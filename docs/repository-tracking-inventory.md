# Repository Tracking Inventory

## Purpose

Records the repository-tracking repair performed as part of SDD `p0-security-fixes` PR1. Documents what was changed, why, and verification evidence.

## Changes

### `.gitignore` replacement

- **Before**: blanket `/frontend` and `/backend` rules excluded all source code from version control
- **After**: targeted exclusions for local secrets, dependency caches, Python tool caches, known build/test output, editor/OS/agent state, and log files
- **Commit**: `7fd6a6a` (2026-07-15)

### `SECURITY.md` added

- Credential rotation runbook for Supabase password and JWT `SECRET_KEY`
- Documented only — rotation is NOT performed by this change
- No credential values contained in the file

## Excluded categories

| Category | Patterns | Evidence |
| ---------- | ---------- | ---------- |
| Local secrets | `.env`, `.env.*`, `!.env.example`, `!.env.template` | `git check-ignore -v .env → .gitignore:2` |
| JS deps | `**/node_modules/`, `**/.npm/`, `**/.cache/` | `git check-ignore -v frontend/node_modules/package.json → .gitignore:6` |
| Python caches | `**/.venv/`, `**/venv/`, `**/__pycache__/`, `**/*.py[cod]`, `.pytest_cache/`, `.mypy_cache/`, `.ruff_cache/` | `git check-ignore -v backend/__pycache__/module.pyc → .gitignore:14` |
| Build outputs | `/frontend/dist/`, `/frontend/coverage/`, `/backend/dist/`, `/backend/build/`, `/backend/coverage/`, `.coverage`, `htmlcov/` | `git check-ignore -v frontend/dist/app.js → .gitignore:20` |
| Editor/OS state | `.idea/`, `.atl/`, `*.swp`, `*.swo`, `.DS_Store`, `Thumbs.db`, `*.log` | `git check-ignore -v .idea/workspace.xml → .gitignore:31` |

## Verification

### Non-destructive cleanup check

```
$ git clean -nd
→ No output — no source or documentation file is eligible for deletion
```

### Committed paths

Source and documentation files committed in this change:

| Path | Status | Category |
| ------ | -------- | ---------- |
| `.gitignore` | modified | configuration |
| `SECURITY.md` | added | documentation |
| `docs/repository-tracking-inventory.md` | added | documentation |

**Total: 3 committed paths** (1 config, 2 docs).

Note: No `frontend/` or `backend/` subdirectories exist in this repository's working tree. The previous blanket `.gitignore` rules were preemptive — no application source code was untracked. The fix ensures that if those directories are created in the future, their source files will be properly tracked.

### Secret scan

No secret scanner executable (`gitleaks`, `trufflehog`) is installed in this environment. Manual review of staged diff confirmed no credential values.

## Date

2026-07-15

## Commit reference

`7fd6a6a67e125e4992593471f0b3310d2fb8ed03` on branch `security/track-source-and-runbook`
