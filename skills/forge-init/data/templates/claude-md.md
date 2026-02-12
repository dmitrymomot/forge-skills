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
- Modern Go 1.25+: `for range n`, `min()`/`max()`, `clear()`, `slices.Contains()`
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
go tool task dev          # Run with hot reload
go tool task build        # Build binary
go tool task test         # Run tests
go tool task lint         # Run linter
go tool task deps         # Update and tidy dependencies
```

<!-- CONDITIONAL_COMMANDS -->
`````

## Subsystem Bullets

Insert one bullet per enabled subsystem into the `<!-- SUBSYSTEM_BULLETS -->` placeholder:

| Subsystem | Bullet |
|-----------|--------|
| db | - **PostgreSQL** — Migrations in `db/migrations/`, sqlc queries in `db/queries/`, generated repo in `internal/repository/`. Config: `db.Config` |
| redis | - **Redis** — Cache and session backend. Config: `forgeredis.Config` |
| sessions | - **Sessions** — `c.AuthenticateSession()`, `c.IsAuthenticated()`, `c.UserID()`. Enabled via `forge.WithSession()` |
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
├── Taskfile.yml
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
go tool task docker:up    # Start Docker services
go tool task docker:down  # Stop Docker services
```
````

### When db is enabled:

````markdown
```bash
go tool task db:migration:create -- <name>  # Create migration
go tool task db:generate                     # Generate sqlc code
```
````

### When any of htmx/alpine/tailwind is enabled:

````markdown
```bash
go tool task assets:download  # Download vendored frontend assets
```
````

### When tailwind is enabled:

````markdown
```bash
go tool task css              # Build Tailwind CSS files
```
````

### When templ is enabled:

````markdown
```bash
go tool task templ:generate  # Compile .templ files to Go
```
````

## Assembly Rules

1. Start with the base template
2. Replace `{{APP_NAME}}` and `{{MODULE_PATH}}` with actual values
3. Insert subsystem bullets for each enabled subsystem
4. Build the project structure tree from the actual generated directories
5. Append conditional command sections for enabled subsystems
6. Remove all `<!-- ... -->` placeholder comments from the final output
