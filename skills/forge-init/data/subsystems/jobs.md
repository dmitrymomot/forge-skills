# Background Jobs Subsystem (River)

## Requires

- `db` (PostgreSQL) — auto-added if not selected

## Imports (config.go)

```go
"github.com/dmitrymomot/forge/pkg/job"
```

## Imports (main.go)

None beyond what `db` subsystem already provides.

## Config Field

```go
Job job.Config `envPrefix:"JOB_"`
```

## Init Code

None — the pool from DB init is reused.

## Health Check

None — job health is managed internally by River via the DB connection.

## App Option

```go
forge.WithJobs(pool, cfg.Job),
```

Note: `pool` comes from the DB subsystem init. This option MUST appear after the DB pool is initialized.

## Shutdown Hook

None — job shutdown is handled internally by the `WithJobs` option.

## Env Vars (dev)

```env
JOB_MAX_WORKERS=5
```

## Env Vars (example)

```env
JOB_MAX_WORKERS=5
```

## Docker Service

None — uses the same PostgreSQL instance as the `db` subsystem.

## Notes

- Jobs require the `db` subsystem. If the user selects jobs without db, auto-add db.
- Creates `internal/tasks/` directory for job handler implementations.
- Workers are registered via `app.RegisterWorker()` or similar pattern.
- The `pool` variable must be available from DB init before `WithJobs` is called.
