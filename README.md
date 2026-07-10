# Auth Monorepo

Google OAuth + password authentication stack.

| Service | Directory | URL (Docker) |
|---------|-----------|--------------|
| Backend API | `google-auth-standalone/` | http://localhost:8090 |
| Frontend SPA | `google-auth-frontend/` | http://localhost:5173 |
| PostgreSQL | (compose service `db`) | localhost:5432 |

## Quick start

```bash
cp google-auth-standalone/.env.example google-auth-standalone/.env
# Edit .env: Google OAuth, SESSION_SECRET, optional SMTP
docker compose up --build
```

## Agent / Cursor setup

- **Guidelines:** [AGENTS.md](./AGENTS.md)
- **Rules:** `.cursor/rules/`
- **Skills:** `.agents/skills/`
- **Bootstrap prompt for new chats:** [docs/AGENT_BOOTSTRAP.md](./docs/AGENT_BOOTSTRAP.md)

## Git remotes

- Backend: `git@github.com:wh1plash/google-auth-standalone.git`
- Frontend: `git@github.com:wh1plash/google-auth-frontend.git`

Each subdirectory has its own git repo. Root compose is local workspace glue.
