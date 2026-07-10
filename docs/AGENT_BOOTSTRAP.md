# Agent Bootstrap Prompt

Copy everything below the line into a new Cursor chat when starting work on this project.

---

## PROMPT START

You are working on the **Auth** monorepo — a standalone Google OAuth + password authentication stack.

### Workspace

```
/home/volodymyr/Oleh/go/projects/Own/Auth/
├── AGENTS.md                    ← read first
├── .cursor/rules/               ← Cursor rules (alwaysApply + file globs)
├── .agents/skills/              ← task-specific skills (read before matching tasks)
├── docker-compose.yml           ← runs full stack
├── google-auth-standalone/      ← Go backend (separate git repo)
└── google-auth-frontend/        ← React SPA (separate git repo)
```

**This is NOT** `/home/volodymyr/Oleh/go/projects/ExistLive/backend-api`. Do not use GORM, Wire, datastore, or Existlive patterns.

### GitHub

| Repo | Remote |
|------|--------|
| Backend | `git@github.com:wh1plash/google-auth-standalone.git` |
| Frontend | `git@github.com:wh1plash/google-auth-frontend.git` |

SSH may use host alias `github.com-wh1plash` in `~/.ssh/config`.

### Product summary

JSON API + React SPA demonstrating:

- Google OAuth 2.0 Authorization Code + OIDC (`go-oidc/v3`, not `google.golang.org/api`)
- Username/password register & login
- Hybrid accounts (Google + password)
- HMAC-signed **Bearer** session tokens in `Authorization` header (cookies removed)
- Admin user management (`role: admin`)
- SMTP emails: registration, credentials added, password reset, set-password
- PostgreSQL schema `"AUTH"`, SQL migrations, auto-run on startup

### Running locally (Docker)

```bash
cd /home/volodymyr/Oleh/go/projects/Own/Auth
cp google-auth-standalone/.env.example google-auth-standalone/.env
docker compose up --build
```

| Service | URL |
|---------|-----|
| Frontend | http://localhost:5173 |
| Backend + Swagger | http://localhost:8090/swagger/ |
| DB | postgres://postgres:nbusr123@localhost:5432/postgres |

### Backend architecture

**Module:** `google-auth-standalone` | **Go:** 1.25 | **Router:** `net/http` ServeMux (Go 1.22+ patterns)

```
cmd/server/main.go
internal/domain/entity/          # User, GoogleProfile — no JSON/DB tags
internal/domain/repository/      # UserRepository interface
internal/usecase/auth/           # login, register, OAuth, credentials, password reset
internal/usecase/user/           # me, list, admin CRUD
internal/adapter/http/           # router, controllers, middleware, request/response, presenter
internal/adapter/oauth/google.go # Google OAuth + OIDC
internal/adapter/persistence/postgres/user_repository.go
internal/infrastructure/
  config/      # env loader
  database/    # pgx pool
  session/     # HMAC Bearer tokens + CSRF state + password reset tokens (1h TTL)
  mail/        # go-pkgz/email SMTP
  migration/   # SQL runner → AUTH.schema_migrations
```

**Dependency rule:** domain ← usecase ← adapters; only `cmd/server` wires everything.

### Account types

| Type | google_id | username/password | Use forgot-password? |
|------|-----------|-------------------|----------------------|
| Google-only | yes | no | yes → **set password** email (username required on reset) |
| Password-only | no | yes | yes → **reset** email |
| Hybrid | yes | yes | yes → reset email |

- `IsConfirmed()` = Google linked
- `HasCredentials()` = username set
- Forgot-password always returns HTTP 200 (anti-enumeration); SMTP failures logged, not exposed

**Recent change (already implemented):** `password_reset.go` supports both reset and set-password flows; `ResetPassword` accepts optional `username`; `SendSetPasswordEmail` in mail.go; frontend `reset-password.tsx` has username field.

### API endpoints (main)

**Public:**
- `GET /api/auth/google/url` → OAuth URL
- `POST /api/auth/google/exchange` → login with code
- `POST /api/auth/register`, `POST /api/auth/login`
- `POST /api/auth/password/forgot`, `POST /api/auth/password/reset`

**Protected (Bearer):**
- `GET /api/me`
- `POST /api/auth/credentials` — add username/password to Google account
- `PATCH /api/auth/credentials/password`, `PATCH /api/auth/credentials/username`
- `POST /api/auth/google/link/*`, `POST /api/auth/google/unlink`
- `GET /api/users`, `PATCH /api/users/{id}`, `DELETE /api/users/{id}` (admin)

### Frontend

- React 19, Vite 8, TanStack Router + Query, Tailwind 4
- Routes: `src/routes/` (login, profile, forgot-password, reset-password, users, auth/callback, …)
- API: `src/lib/api.ts` — `VITE_API_BASE_URL` (Docker: build arg, default `http://localhost:8090`)
- Session: `src/lib/auth-storage.ts` (localStorage)
- Query keys must include session token where user-specific data is fetched

### Docker specifics

- Root compose builds `google-auth-standalone/` and `google-auth-frontend/`
- Backend Dockerfile: `go build -mod=vendor` — **no network in build**; run `go mod vendor` after dep changes
- Backend env from `google-auth-standalone/.env` via `env_file`
- SMTP: `SMTP_HOST`, `SMTP_PORT` (465=TLS, 587=STARTTLS), `SMTP_USER`, `SMTP_PASSWORD`, `MAIL_FROM`
- `FRONTEND_URL` used in email links (default `http://localhost:5173`)

### Agent rules (must follow)

1. Read `AGENTS.md` and relevant `.agents/skills/*/SKILL.md` before tasks.
2. **Minimal diff** — no unrelated refactors.
3. **No git commits** unless I explicitly ask.
4. **Never commit `.env`** or secrets.
5. **Never manually edit** `google-auth-standalone/docs/*` — run `make swagger`.
6. Code/comments in English; explain to me in Russian if I write in Russian.
7. Add table-driven tests for non-trivial usecase logic.

### Skills map

| Task | Skill path |
|------|------------|
| REST endpoint | `.agents/skills/auth-add-endpoint/SKILL.md` |
| DB migration | `.agents/skills/auth-db-migrations/SKILL.md` |
| Docker / compose | `.agents/skills/auth-docker/SKILL.md` |
| Frontend page | `.agents/skills/auth-frontend-feature/SKILL.md` |

### Dev commands

```bash
# Backend
cd google-auth-standalone
make run | make swagger | make migrate | go test ./...

# Frontend
cd google-auth-frontend
npm run dev | npm run build | npm run routes:generate

# Full stack
cd /home/volodymyr/Oleh/go/projects/Own/Auth
docker compose up --build
```

### Known pitfalls

- `vendor/` may contain upstream `appveyor.yml` with Slack webhooks → GitHub push protection; delete before push.
- Google-only users won't get reset emails unless set-password flow is used (now implemented).
- `VITE_*` changes need frontend image rebuild.
- After `go commit --amend` post-push → `git push --force-with-lease`.

### Current task

<!-- Replace this section with your actual task when pasting -->

[Describe what you want to build/fix here]

## PROMPT END
