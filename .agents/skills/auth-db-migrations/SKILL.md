---
name: auth-db-migrations
description: Create SQL up/down migrations for google-auth-standalone using scripts/create-migration.sh. Schema AUTH, tracked in AUTH.schema_migrations. Use for table/column/index changes.
---

# Database Migrations

## When to use

- Schema changes for PostgreSQL `AUTH` schema
- New tables, columns, constraints, indexes

## Hard rules

1. Every migration is a **pair**: `.up.sql` + `.down.sql`.
2. Create files via `scripts/create-migration.sh` — do not invent timestamps manually.
3. Tables live in `"AUTH"` schema (quoted, uppercase).
4. No seed data in migrations.
5. Migrations auto-run on server start; manual apply: `make migrate`.

## Procedure

### 1. Create migration files

```bash
cd google-auth-standalone
./scripts/create-migration.sh "describe_change"
```

Creates `db/migrations/YYYYMMDDHHMMSS_describe_change.up.sql` and `.down.sql`.

### 2. Write SQL

**up.sql** — forward change:

```sql
ALTER TABLE "AUTH".users ADD COLUMN IF NOT EXISTS example TEXT;
```

**down.sql** — safe rollback:

```sql
ALTER TABLE "AUTH".users DROP COLUMN IF EXISTS example;
```

### 3. Update repository if needed

- `internal/adapter/persistence/postgres/user_repository.go`
- Domain entity: `internal/domain/entity/user.go`

### 4. Verify

```bash
make migrate
go test ./internal/infrastructure/migration/...
```

## Docker

Migrations copied into image at `db/migrations`. Rebuild backend after new migration files:

```bash
docker compose build backend
```

## Registry

Applied migrations recorded in `"AUTH".schema_migrations`. Runner: `internal/infrastructure/migration/runner.go`.
