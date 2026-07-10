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

## Git layout

This directory is the **workspace root** repo. Backend and frontend are **git submodules**:

| Path | Remote |
|------|--------|
| `google-auth-standalone/` | `git@github.com:wh1plash/google-auth-standalone.git` |
| `google-auth-frontend/` | `git@github.com:wh1plash/google-auth-frontend.git` |

### Clone (first time)

```bash
git clone --recurse-submodules git@github.com:wh1plash/google-auth-workspace.git Auth
cd Auth
```

Or after a plain clone:

```bash
git submodule update --init --recursive
```

### Push workspace root to GitHub

Create an empty repo `google-auth-workspace` on GitHub, then:

```bash
git remote add origin git@github.com-wh1plash:wh1plash/google-auth-workspace.git
git push -u origin main
```

### After submodule changes

```bash
cd google-auth-standalone && git commit && git push
cd ../google-auth-frontend && git commit && git push
cd .. && git add google-auth-standalone google-auth-frontend
git commit -m "chore: bump submodules"
git push
```
