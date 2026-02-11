# PostgreSQL Database Subsystem

## Imports (config.go)

```go
"github.com/dmitrymomot/forge/pkg/db"
```

## Imports (main.go)

```go
"github.com/dmitrymomot/forge/pkg/db"
dbmigrations "{{MODULE_PATH}}/db/migrations"
```

Note: no `"embed"` import needed in main.go — the embed directive lives in the `db/migrations` package.

## Config Field

```go
DB db.Config `envPrefix:"DB_"`
```

## Init Code

```go
pool, err := db.Open(ctx, cfg.DB, db.WithMigrations(dbmigrations.FS))
if err != nil {
    log.Fatal("failed to connect to database: ", err)
}
```

## Health Check

```go
forge.HealthCheck("db", db.Healthcheck(pool)),
```

## App Option

None — DB is used via the pool passed to handlers and other subsystems.

## Shutdown Hook

```go
forge.WithShutdownHook(db.Shutdown(pool)),
```

## Generated Files

When `db` is selected, the following files are generated in addition to code in `cmd/`:

| File                              | Template reference       | Purpose                                  |
|-----------------------------------|--------------------------|------------------------------------------|
| `db/sqlc.yml`                     | `templates/sqlc.md`      | sqlc configuration                       |
| `db/migrations/embed.go`         | `templates/db-init.md`   | Embed package exposing migration FS      |
| `db/migrations/00001_init.sql`   | `templates/db-init.md`   | Initial migration (updated_at trigger)   |
| `db/queries/.gitkeep`            | —                        | Placeholder for sqlc query files         |
| `internal/repository/.gitkeep`   | —                        | Placeholder for sqlc output              |

## go.mod Tool Addition

When `db` is selected, add sqlc to the tool directive:

```
github.com/sqlc-dev/sqlc/cmd/sqlc
```

## Env Vars (dev)

```env
DB_URL=postgres://postgres:postgres@localhost:5432/{{APP_NAME}}?sslmode=disable
```

## Env Vars (example)

```env
DB_URL=postgres://user:password@localhost:5432/dbname?sslmode=disable
```

## Docker Service

```yaml
postgres:
  image: postgres:18-alpine
  environment:
    POSTGRES_USER: postgres
    POSTGRES_PASSWORD: postgres
    POSTGRES_DB: {{APP_NAME}}
  ports:
    - "5432:5432"
  volumes:
    - postgres_data:/var/lib/postgresql/data
  healthcheck:
    test: ["CMD-SHELL", "pg_isready -U postgres"]
    interval: 5s
    timeout: 5s
    retries: 5
```

## Docker Volume

```yaml
postgres_data:
```
