# PostgreSQL Database Subsystem

## Imports (config.go)

```go
"github.com/dmitrymomot/forge/pkg/db"
```

## Imports (main.go)

```go
"embed"

"github.com/dmitrymomot/forge/pkg/db"
```

Note: the `"embed"` import may be shared with other subsystems (frontend static files). Deduplicate when assembling.

## Config Field

```go
DB db.Config `envPrefix:"DB_"`
```

## Embed Directive

Add this at package level in main.go, before the `main()` function:

```go
//go:embed migrations
var migrationsFS embed.FS
```

Note: if frontend static embed is also present, both embed directives coexist. The `"embed"` import is only needed once.

## Init Code

```go
pool, err := db.Open(ctx, cfg.DB, db.WithMigrations(migrationsFS))
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
