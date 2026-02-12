# Sessions Subsystem

## Requires

`db` — PostgreSQL-backed session store.

## Imports (config.go)

No additional imports needed — cookie config is already in the base.

## Imports (main.go)

```go
"time"

"{{MODULE_PATH}}/internal/repository"
"{{MODULE_PATH}}/internal/sessionstore"
```

## Config Field

No additional config fields — `cookie.Config` is already in the base config.

## Init Code

```go
repo := repository.New(pool)
sessionStore := sessionstore.New(repo)
```

**Note:** `pool` comes from the `db` subsystem init code. Because `sessions` requires `db`, the pool is always available. Place this code after the db init block.

## Health Check

None.

## App Option

```go
forge.WithSession(sessionStore, forge.WithSessionTTL(7*24*time.Hour)),
```

## Shutdown Hook

None.

## Env Vars (dev)

Included in base config — see `templates/env-vars.md` base Cookie section.

## Env Vars (example)

Included in base config — see `templates/env-vars.md` base Cookie section.

## Docker Service

None.

## Generated Files

When `sessions` is selected, generate the following files from `templates/sessions-init.md`:

| File | Purpose |
|------|---------|
| `db/migrations/00002_users_and_sessions.sql` | Users + sessions tables with indexes |
| `db/queries/sessions.sql` | sqlc queries for all `forge.SessionStore` methods |
| `internal/sessionstore/store.go` | Go adapter implementing `forge.SessionStore` |

These files are created during **Phase 1** (directory structure creation).

## Notes

- Cookie config fields (`COOKIE_*`) are part of the base config and always present.
- When sessions is enabled, the env values become meaningful (especially `COOKIE_SECRET`).
- The session store is backed by PostgreSQL via sqlc-generated repository code.
- `go tool sqlc generate -f db/sqlc.yml` must run after file generation to produce the repository package.
- Available session methods on `forge.Context`: `c.AuthenticateSession()`, `c.IsAuthenticated()`, `c.UserID()`, `c.DestroySession()`.
