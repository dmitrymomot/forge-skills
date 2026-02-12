# Sessions Subsystem

## Imports (config.go)

No additional imports needed — cookie config is already in the base.

## Imports (main.go)

```go
"time"
```

## Config Field

No additional config fields — `cookie.Config` is already in the base config.

## Init Code

```go
// TODO: Provide your own session store implementation.
// Example stores: cookie-based, Redis-backed, DB-backed.
// The store must implement the forge.SessionStore interface.
var sessionStore forge.SessionStore // replace with your implementation
```

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

## Notes

- Cookie config fields (`COOKIE_*`) are part of the base config and always present.
- When sessions is enabled, the env values become meaningful (especially `COOKIE_SECRET`).
- The session store is intentionally a TODO — the user should implement or choose a store based on their stack (Redis store, DB store, etc.).
- Available session methods on `forge.Context`: `c.AuthenticateSession()`, `c.IsAuthenticated()`, `c.UserID()`, `c.DestroySession()`.
