---
description: Core agent guidelines for the Auth monorepo (google-auth-standalone + google-auth-frontend)
alwaysApply: true
---

# Auth Monorepo Agent Rules

Read `/home/volodymyr/Oleh/go/projects/Own/Auth/AGENTS.md` for full project context.

## Quick reference

- **Root:** `/home/volodymyr/Oleh/go/projects/Own/Auth`
- **Backend:** `google-auth-standalone/` — Go 1.25, Clean Architecture, module `google-auth-standalone`
- **Frontend:** `google-auth-frontend/` — React 19, TanStack Router/Query, Vite
- **Compose:** root `docker-compose.yml` builds both services

## Hard rules

1. Do **not** confuse this project with ExistLive/backend-api.
2. Do **not** manually edit `google-auth-standalone/docs/*` — run `make swagger`.
3. Do **not** commit `.env` files.
4. Keep controllers thin; business logic in `internal/usecase`.
5. After `go.mod` changes: `go mod tidy && go mod vendor` (Docker uses `-mod=vendor`).
6. Only create git commits when the user explicitly requests.

## Skills (read before task)

| Task | Skill |
|------|-------|
| New/changed REST endpoint | `.agents/skills/auth-add-endpoint/SKILL.md` |
| DB schema change | `.agents/skills/auth-db-migrations/SKILL.md` |
| Docker / compose | `.agents/skills/auth-docker/SKILL.md` |
| Frontend route/page | `.agents/skills/auth-frontend-feature/SKILL.md` |

## Planning mode

Challenge requirements when needed; propose simpler solutions; follow YAGNI and minimal diff.
