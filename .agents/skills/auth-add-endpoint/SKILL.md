---
name: auth-add-endpoint
description: Add or modify a REST endpoint in google-auth-standalone — controller handler, router registration, request/response DTOs, presenter if needed, Swagger godoc, make swagger. Use when adding routes, handlers, or API changes.
---

# Add / Modify REST Endpoint

## When to use

- New or changed HTTP handler in `google-auth-standalone`
- Route registration, Swagger docs, request/response shapes

## Hard rules

1. **Never manually edit** `google-auth-standalone/docs/*` — run `make swagger`.
2. Controllers stay **thin**: decode JSON → usecase → presenter → `httpx.WriteJSON`.
3. Domain entities never appear in JSON — use `response.*` DTOs via presenter.
4. Register routes in `internal/adapter/http/router.go` with `METHOD /path` syntax.
5. Protected routes wrap with `authMW.RequireAPI`; admin routes use `adminOnly`.

## Procedure

### 1. Request/response DTOs

- Inbound: `internal/adapter/http/request/`
- Outbound: `internal/adapter/http/response/`
- Add `json` tags and swagger `example:` where appropriate.

### 2. Usecase method

- Business logic in `internal/usecase/auth/` or `internal/usecase/user/`.
- Return domain entities or sentinel errors.

### 3. Controller handler

- File: `internal/adapter/http/controller/`
- Add Swagger godoc block (`@Summary`, `@Router`, `@Tags`, `@Param`, `@Success`, `@Failure`).
- Map usecase errors to HTTP status (400, 403, 409, etc.).

### 4. Router

- Add `mux.HandleFunc` or `mux.Handle` in `router.go`.
- Place in public vs protected section correctly.

### 5. Regenerate Swagger

```bash
cd google-auth-standalone
make swagger
```

### 6. Verify

```bash
go build ./...
go test ./internal/usecase/...   # if tests exist
```

## Frontend follow-up

If the SPA consumes the endpoint, update `google-auth-frontend/src/lib/api.ts` and relevant route/component. See `auth-frontend-feature` skill.

## File index

| Layer | Path |
|-------|------|
| Router | `internal/adapter/http/router.go` |
| Controllers | `internal/adapter/http/controller/` |
| DTOs | `internal/adapter/http/request/`, `response/` |
| Presenter | `internal/adapter/http/presenter/` |
| Usecase | `internal/usecase/auth/`, `user/` |
