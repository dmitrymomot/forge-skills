# Base Application Templates

These templates form the foundation of every generated Forge app. Subsystem-specific code is injected at marked insertion points.

---

## go.mod Requirements

The generated `go.mod` must specify Go 1.25 as the minimum version:

```
go 1.25
```

### Tool Directive

Go 1.25+ supports a `tool (...)` directive in go.mod for dev tool dependencies. Include the following base tools in every project:

```
tool (
	github.com/air-verse/air
	github.com/dkorunic/betteralign/cmd/betteralign
	github.com/go-task/task/v3/cmd/task
	github.com/golangci/golangci-lint/v2/cmd/golangci-lint
	github.com/pressly/goose/v3/cmd/goose
	go.uber.org/nilaway/cmd/nilaway
	golang.org/x/tools/cmd/goimports
	golang.org/x/tools/go/analysis/passes/modernize/cmd/modernize
)
```

**Conditional tool additions:**

| Condition       | Tool to add                                    |
|-----------------|------------------------------------------------|
| templ selected  | `github.com/a-h/templ/cmd/templ`               |
| templ selected  | `github.com/templui/templui/cmd/templui`        |
| db selected     | `github.com/sqlc-dev/sqlc/cmd/sqlc`             |

After writing go.mod with `go mod init`, add the tool directive block, then run `go mod tidy`.

---

## Import Injection for Config Types

Subsystem config types are imported from their actual packages, not from the `forge` root. When assembling `config.go`, inject these imports based on enabled subsystems:

| Subsystem | Import path                                     | Config type used as      |
|-----------|--------------------------------------------------|--------------------------|
| db        | `"github.com/dmitrymomot/forge/pkg/db"`          | `db.Config`              |
| redis     | `forgeredis "github.com/dmitrymomot/forge/pkg/redis"` | `forgeredis.Config` |
| jobs      | `"github.com/dmitrymomot/forge/pkg/job"`         | `job.Config`             |
| storage   | `"github.com/dmitrymomot/forge/pkg/storage"`     | `storage.Config`         |
| mailer    | `"github.com/dmitrymomot/forge/pkg/mailer"` + `"github.com/dmitrymomot/forge/pkg/mailer/smtp"` | `mailer.Config` + `smtp.Config` |
| oauth     | `"github.com/dmitrymomot/forge/pkg/oauth"`       | `oauth.GoogleConfig` + `oauth.GitHubConfig` |

Base config types (`forge.AppConfig`, `forge.RunConfig`, `forge.CookieConfig`) remain in the `forge` package. Middleware config types (`middlewares.CORSConfig`) remain in the `middlewares` package.

---

## config.go

```go
package main

import (
	"github.com/dmitrymomot/forge"
	"github.com/dmitrymomot/forge/middlewares"
	// {{SUBSYSTEM_IMPORTS}}
)

type Config struct {
	App    forge.AppConfig        `envPrefix:"APP_"`
	Run    forge.RunConfig        `envPrefix:"RUN_"`
	Cookie forge.CookieConfig     `envPrefix:"COOKIE_"`
	CORS   middlewares.CORSConfig `envPrefix:"CORS_"`
	// {{SUBSYSTEM_CONFIG_FIELDS}}
}
```

### Insertion Points

- `{{SUBSYSTEM_IMPORTS}}`: Additional import paths required by subsystem config types
- `{{SUBSYSTEM_CONFIG_FIELDS}}`: Struct fields for each enabled subsystem

---

## main.go — Single-domain mode (`app.Run()`)

```go
package main

import (
	"context"
	"log"

	"github.com/dmitrymomot/forge"
	"github.com/dmitrymomot/forge/middlewares"
	// {{SUBSYSTEM_IMPORTS}}
)

func main() {
	var cfg Config
	if err := forge.LoadConfig(&cfg); err != nil {
		log.Fatal("failed to load config: ", err)
	}

	ctx := context.Background()

	// {{SUBSYSTEM_INIT}}

	app := forge.New(cfg.App,
		forge.WithLogger("{{APP_NAME}}", forge.RequestIDExtractor()),
		forge.WithCookieConfig(cfg.Cookie),
		forge.WithMiddleware(
			middlewares.RequestID(middlewares.RequestIDConfig{}),
			middlewares.Recover(middlewares.RecoverConfig{}),
			middlewares.CORS(cfg.CORS),
		),
		forge.WithHealthChecks(
			// {{SUBSYSTEM_HEALTH_CHECKS}}
		),
		// {{SUBSYSTEM_APP_OPTIONS}}
	)

	// Register route groups
	// app.Register(&handler.MyHandler{})

	if err := app.Run(cfg.Run,
		// {{SUBSYSTEM_SHUTDOWN_HOOKS}}
	); err != nil {
		log.Fatal(err)
	}

	_ = ctx // used by subsystem init code
}
```

---

## main.go — Multi-domain mode (`forge.Run()`)

```go
package main

import (
	"context"
	"log"

	"github.com/dmitrymomot/forge"
	"github.com/dmitrymomot/forge/middlewares"
	// {{SUBSYSTEM_IMPORTS}}
)

func main() {
	var cfg Config
	if err := forge.LoadConfig(&cfg); err != nil {
		log.Fatal("failed to load config: ", err)
	}

	ctx := context.Background()

	// {{SUBSYSTEM_INIT}}

	app := forge.New(cfg.App,
		forge.WithLogger("{{APP_NAME}}", forge.RequestIDExtractor()),
		forge.WithCookieConfig(cfg.Cookie),
		forge.WithMiddleware(
			middlewares.RequestID(middlewares.RequestIDConfig{}),
			middlewares.Recover(middlewares.RecoverConfig{}),
			middlewares.CORS(cfg.CORS),
		),
		forge.WithHealthChecks(
			// {{SUBSYSTEM_HEALTH_CHECKS}}
		),
		// {{SUBSYSTEM_APP_OPTIONS}}
	)

	// Register route groups
	// app.Register(&handler.MyHandler{})

	if err := forge.Run(cfg.Run,
		forge.WithFallback(app),
		// {{SUBSYSTEM_SHUTDOWN_HOOKS}}
	); err != nil {
		log.Fatal(err)
	}

	_ = ctx // used by subsystem init code
}
```

---

## Insertion Point Reference

| Placeholder                    | Location     | Description                                      |
|-------------------------------|--------------|--------------------------------------------------|
| `{{APP_NAME}}`                | main.go      | App name for logger                              |
| `{{SUBSYSTEM_IMPORTS}}`       | both files   | Import paths for enabled subsystems              |
| `{{SUBSYSTEM_CONFIG_FIELDS}}` | config.go    | Config struct fields                             |
| `{{SUBSYSTEM_INIT}}`         | main.go      | Initialization code (db.Open, redis.Open, etc.)  |
| `{{SUBSYSTEM_HEALTH_CHECKS}}`| main.go      | Health check registrations                       |
| `{{SUBSYSTEM_APP_OPTIONS}}`  | main.go      | forge.With* options                              |
| `{{SUBSYSTEM_SHUTDOWN_HOOKS}}`| main.go     | Shutdown hooks for graceful cleanup              |

### Assembly Rules

1. Collect all sections from each enabled subsystem's data file
2. Merge imports (deduplicate) — include config type imports per the Import Injection table above
3. Place config fields in order: db, redis, jobs, storage, mailer, oauth
4. Place init code in dependency order: db first (needed by jobs), redis, storage, mailer, oauth
5. Health checks and shutdown hooks follow the same order
6. Assemble embed-based imports via dedicated packages (no inline `//go:embed` directives in main.go):
   - **DB**: If `db` is enabled, add `dbmigrations "{{MODULE_PATH}}/db/migrations"` to main.go imports. Use `dbmigrations.FS` in init code. No `"embed"` import needed.
   - **Frontend**: If ANY frontend option (`htmx`, `alpine`, `tailwind`) is enabled, add `"{{MODULE_PATH}}/assets"` to main.go imports. Use `forge.WithStaticFiles("/static/", assets.StaticFS, "static")` in app options. No `"embed"` import needed.
   - These import-based embeds replace the previous `{{EMBED_DIRECTIVES}}` placeholder pattern. The `//go:embed` directives live in their respective packages (`db/migrations/embed.go` and `assets/embed.go`), not in `cmd/main.go`.
7. Remove the `_ = ctx` line if any subsystem init code uses `ctx`
8. Remove trailing commas and empty comment blocks if no subsystems are enabled
