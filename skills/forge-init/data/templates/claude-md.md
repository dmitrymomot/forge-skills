# CLAUDE.md — Project Context for Claude Code

Generate this file at the project root so Claude Code understands the Forge project immediately.

## Base Template (always included)

Replace `{{APP_NAME}}` with the actual app name and `{{MODULE_PATH}}` with the Go module path.

`````markdown
# {{APP_NAME}}

**Module:** `{{MODULE_PATH}}`
**Framework:** [Forge](https://github.com/dmitrymomot/forge) — Go micro-SaaS framework

## Forge Framework

Core types: `forge.App`, `forge.Context`, `forge.Router`, `forge.Handler`, `forge.HandlerFunc`, `forge.Middleware`

`forge.New()` creates an app, `forge.Run()` starts the server. `forge.Context` embeds `context.Context` — pass directly to DB calls, HTTP clients, etc.

Handler pattern:

```go
type MyHandler struct{ repo *repository.Queries }

func (h *MyHandler) Routes(r forge.Router) {
    r.GET("/items", h.list)
    r.POST("/items", h.create)
}

func (h *MyHandler) list(c forge.Context) error {
    return c.JSON(200, items)
}
```

Type-safe params: `forge.Param[int64](c, "id")`, `forge.QueryDefault[int](c, "page", 1)`

Data binding/validation uses semicolon-separated tags: `validate:"required;max:100"` / `sanitize:"trim;numeric"`

## Design Rules

- No reflection, no service containers, no magic
- Packages receive values via parameters, not context
- All IDs via `pkg/id/` — never `uuid.New()` or `ulid.Make()`
- Modern Go 1.26+: `for range n`, `min()`/`max()`, `clear()`, `slices.Contains()`, `errors.AsType[T]()`, `reflect.Value.Fields()`
- Validator/sanitizer tags use semicolons as separators, colons for params
- `go build -o /dev/null ./...` — never leave binary artifacts
- `requestContext` session methods are NOT goroutine-safe
- No `v := v` captures (Go 1.22+ loop variables)

## Enabled Subsystems

<!-- SUBSYSTEM_BULLETS -->

## Project Structure

<!-- PROJECT_STRUCTURE -->

## Common Commands

```bash
just dev          # Run with hot reload
just build        # Build binary
just test         # Run tests
just lint         # Run linter
just check        # Format, lint, and test (pre-commit)
just clean        # Remove build artifacts and caches
just deps         # Update and tidy dependencies
```

<!-- CONDITIONAL_COMMANDS -->
`````

## Subsystem Bullets

Insert one bullet per enabled subsystem into the `<!-- SUBSYSTEM_BULLETS -->` placeholder:

| Subsystem | Bullet |
|-----------|--------|
| db | - **PostgreSQL** — Migrations in `db/migrations/`, sqlc queries in `db/queries/`, generated repo in `internal/repository/`. Config: `db.Config` |
| redis | - **Redis** — Cache and session backend. Config: `forgeredis.Config` |
| sessions | - **Sessions** — PostgreSQL-backed session store in `internal/sessionstore/`. `c.AuthenticateSession()`, `c.IsAuthenticated()`, `c.UserID()`. Enabled via `forge.WithSession()` |
| jobs | - **Jobs** — River-based background jobs. `c.Enqueue("task_name", payload)`. Workers in `internal/worker/`. Config: `job.Config` |
| storage | - **Storage** — S3-compatible file storage. `c.Upload()`, `c.FileURL()`. Config: `storage.Config` |
| templ | - **templ** — Type-safe HTML templates. Run `go tool templ generate` to compile `.templ` files |
| htmx | - **HTMX** — `c.IsHTMX()`, `c.RenderPartial(fullPage, partial)`. Vendored JS in `assets/static/js/` |
| alpine | - **Alpine.js** — Lightweight JS framework. Vendored in `assets/static/js/` |
| tailwind | - **TailwindCSS** — Utility CSS framework. CLI-compiled CSS in `assets/static/css/`. Source configs in `assets/src/` |
| mailer | - **Email** — SMTP mailer (Mailpit in dev). Templates in `templates/emails/`, layouts in `templates/emails/layouts/`. Config: `mailer.Config` + `smtp.Config` |
| oauth | - **OAuth** — Google and GitHub providers. Config: `oauth.GoogleConfig` + `oauth.GitHubConfig` |

If no subsystems are enabled, replace the placeholder with: `No optional subsystems enabled.`

## Project Structure

Replace `<!-- PROJECT_STRUCTURE -->` with a directory tree matching the actual generated structure. Build this from the base tree plus conditional directories for each enabled subsystem. Example for a project with db, redis, and htmx:

````markdown
```
./
├── cmd/
│   ├── main.go
│   └── config.go
├── db/
│   ├── sqlc.yml
│   ├── migrations/
│   │   ├── embed.go
│   │   └── 00001_init.sql
│   └── queries/
├── internal/
│   ├── handler/
│   └── repository/
├── assets/
│   ├── embed.go
│   └── static/
│       ├── css/
│       ├── js/
│       └── img/
├── .air.toml
├── .env
├── .env.example
├── .gitignore
├── CLAUDE.md
├── justfile
├── docker-compose.yml
├── go.mod
└── go.sum
```
````

Adjust the tree to reflect only the directories and files that were actually created. Omit directories that weren't generated.

## Conditional Commands

Replace `<!-- CONDITIONAL_COMMANDS -->` with additional command blocks based on enabled subsystems. Only include sections for enabled subsystems:

### When Docker services exist (db, redis, or storage):

````markdown
```bash
just docker-up    # Start Docker services
just docker-down  # Stop Docker services
```
````

### When db is enabled:

````markdown
```bash
just db-migration-create <name>  # Create migration
just db-generate                  # Generate sqlc code
just db-migrate                   # Run pending migrations
just db-migrate-down              # Rollback last migration
just db-status                    # Show migration status
just db-reset                     # Drop + re-migrate (with confirmation)
```
````

### When any of htmx/alpine/tailwind is enabled:

````markdown
```bash
just assets-download  # Download vendored frontend assets
```
````

### When tailwind is enabled:

````markdown
```bash
just css              # Build Tailwind CSS files
```
````

### When templ is enabled:

````markdown
```bash
just templ-generate  # Compile .templ files to Go
```
````

### When db OR templ is enabled:

````markdown
```bash
just generate         # Run all code generation (sqlc + templ)
```
````

Only list the generators that are actually enabled in the parenthetical. E.g., if only db: `(sqlc)`. If only templ: `(templ)`. If both: `(sqlc + templ)`.

### When any Docker service exists (integration tests):

````markdown
```bash
just test-integration  # Run integration tests (or: just ti)
```
````

## Assembly Rules

1. Start with the base template
2. Replace `{{APP_NAME}}` and `{{MODULE_PATH}}` with actual values
3. Insert subsystem bullets for each enabled subsystem
4. Build the project structure tree from the actual generated directories
5. Append conditional command sections for enabled subsystems
6. Remove all `<!-- ... -->` placeholder comments from the final output
