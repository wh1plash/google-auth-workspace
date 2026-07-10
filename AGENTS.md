# Auth Monorepo — Agent Guidelines

## Workspace

**Root:** `/home/volodymyr/Oleh/go/projects/Own/Auth`

| Component | Path | Git remote |
|-----------|------|------------|
| Backend API | `google-auth-standalone/` | `git@github.com:wh1plash/google-auth-standalone.git` |
| Frontend SPA | `google-auth-frontend/` | `git@github.com:wh1plash/google-auth-frontend.git` |
| Docker stack | `docker-compose.yml` (root) | — |

This is **not** ExistLive/backend-api. Do not apply Existlive patterns (GORM, Wire, datastore layer).

## Product

Standalone Google OAuth + password auth demo:

- Go JSON API (Chi-style `http.ServeMux` routing)
- React 19 SPA (Vite, TanStack Router/Query, Tailwind 4)
- PostgreSQL schema `AUTH`
- HMAC **Bearer** session tokens (no cookies)
- Google OAuth via `golang.org/x/oauth2` + `github.com/coreos/go-oidc/v3`
- SMTP transactional mail via `github.com/go-pkgz/email`

## Architecture (backend)

```
cmd/server          → wires everything
internal/domain     → entity + repository interfaces (no I/O)
internal/usecase    → auth.Service, user.Service
internal/adapter    → http controllers, postgres repo, google oauth
internal/infrastructure → config, database, session, mail, migration
```

**Dependency rule:** domain ← usecase ← adapter/infrastructure; only `cmd/server` composes all.

Domain entities have **no** JSON/DB tags. HTTP DTOs live in `adapter/http/request` and `response`; mapping in `presenter/`.

## Account model

| Type | google_id | username/password | confirmed |
|------|-----------|-------------------|-----------|
| Google-only | yes | no | yes (after Google login) |
| Password-only | no | yes | no until Google linked |
| Hybrid | yes | yes | yes |

- `IsConfirmed()` = `GoogleID != ""`
- `HasCredentials()` = `Username != ""`
- Password reset: existing credentials → reset; Google-only confirmed → set password (username required)

## Key API endpoints

Public: `/api/auth/google/*`, `/api/auth/register`, `/api/auth/login`, `/api/auth/password/forgot`, `/api/auth/password/reset`

Protected (Bearer): `/api/me`, `/api/auth/credentials`, `/api/auth/google/link/*`, `/api/users` (admin)

Swagger: `http://localhost:8090/swagger/` — regenerate with `make swagger`, **never edit** `docs/*` manually.

## Frontend

- Routes in `src/routes/` (TanStack file-based routing)
- API client: `src/lib/api.ts` — `VITE_API_BASE_URL` baked at Docker build
- Session: `src/lib/auth-storage.ts` (localStorage Bearer token)

## Docker

From repo root:

```bash
cp google-auth-standalone/.env.example google-auth-standalone/.env  # fill secrets
docker compose up --build
```

Ports: frontend `:5173`, backend `:8090`, db `:5432`

Backend builds with `-mod=vendor` (no `go mod download` in Docker). After dependency changes: `go mod tidy && go mod vendor` in `google-auth-standalone/`.

## Development commands

**Backend** (`google-auth-standalone/`):

```bash
make run          # local server
make swagger      # regen docs
make migrate      # apply migrations
go test ./...
```

**Frontend** (`google-auth-frontend/`):

```bash
npm run dev
npm run build
npm run routes:generate   # after new route files
```

## Agent behavior

1. **Minimal scope** — smallest correct diff; no unrelated refactors.
2. **No commits** unless the user explicitly asks.
3. **Secrets** — never commit `.env`; warn if asked to commit credentials.
4. **Code language** — English for code, comments, commit messages; user may prefer Russian for explanations.
5. **Tests** — add table-driven unit tests for usecase changes when behavior is non-trivial.
6. **Skills** — read relevant `.agents/skills/*/SKILL.md` before endpoint/migration/docker/frontend tasks.

## SSH / Git

GitHub user: `wh1plash`. SSH host alias `github.com-wh1plash` may be configured in `~/.ssh/config` for a dedicated key.

## Known pitfalls

- `vendor/` may contain upstream CI files with secrets — remove `appveyor.yml` etc. before push if GitHub blocks.
- SMTP errors on forgot-password are logged but API returns 200 (anti-enumeration).
- Frontend `VITE_*` vars require image rebuild, not just container restart.
