---
name: auth-docker
description: Build and run the Auth stack with root docker-compose.yml — backend vendor build, frontend Vite args, env files, SMTP, healthchecks. Use for Docker, compose, or deployment issues.
---

# Docker Stack

## When to use

- `docker compose up --build`
- Dockerfile changes, env wiring, port mapping
- Build failures (`go mod download`, vendor, TLS)

## Layout

```
/home/volodymyr/Oleh/go/projects/Own/Auth/
├── docker-compose.yml          # orchestration
├── google-auth-standalone/
│   ├── Dockerfile              # -mod=vendor build
│   ├── .env                    # secrets (gitignored)
│   └── .env.example
└── google-auth-frontend/
    ├── Dockerfile              # node build → nginx
    └── nginx.conf
```

## First-time setup

```bash
cd /home/volodymyr/Oleh/go/projects/Own/Auth
cp google-auth-standalone/.env.example google-auth-standalone/.env
# Fill: GOOGLE_CLIENT_ID, GOOGLE_CLIENT_SECRET, SESSION_SECRET, SMTP_*
docker compose up --build
```

## Services

| Service | Host port | Notes |
|---------|-----------|-------|
| frontend | 5173 | nginx → SPA |
| backend | 8090 | maps to container :8080 |
| db | 5432 | postgres/postgres, password nbusr123 |

## Env mapping (backend)

From `env_file: google-auth-standalone/.env` plus compose overrides:

- `DATABASE_URL` → `postgres://...@db:5432/...`
- `FRONTEND_URL` → reset email links
- `ALLOWED_ORIGINS` → CORS
- `GOOGLE_REDIRECT_URL` → SPA callback

SMTP vars (`SMTP_HOST`, `SMTP_PORT`, `SMTP_USER`, `SMTP_PASSWORD`, `MAIL_FROM`) come **only** from `.env`.

## Frontend build arg

```yaml
VITE_API_BASE_URL: ${VITE_API_BASE_URL:-http://localhost:8090}
```

Changing this requires `docker compose build frontend`, not just restart.

## Backend vendor build

Dockerfile uses `go build -mod=vendor`. After dependency changes:

```bash
cd google-auth-standalone
go mod tidy && go mod vendor
```

## Troubleshooting

| Symptom | Fix |
|---------|-----|
| `go mod download` TLS timeout | Use vendor (already configured) |
| GitHub push blocked (secrets in vendor) | `find vendor -name appveyor.yml -delete` |
| Mail not sent | Check `.env` SMTP; logs: `docker compose logs backend \| grep mail` |
| CORS errors | `ALLOWED_ORIGINS=http://localhost:5173` |

## Healthcheck

Backend: `GET /healthz` via wget inside container.
