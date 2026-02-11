---
name: forge-init
description: "Initialize a new Forge framework application. Use when starting a new project, scaffolding a Forge app, or setting up a Go micro-SaaS backend. Triggers on: new forge app, init forge, scaffold forge, create forge project."
argument-hint: <app-name>
context: fork
allowed-tools: Read, Glob, Grep, Write, Bash
---

# Forge Project Initializer

You are scaffolding a new Forge framework application. Follow the phases below precisely. Read each referenced data file before executing that phase.

The app name from `$ARGUMENTS` (if provided) should be used as the default app name.

---

## Phase 0: Gather Requirements

### Directory Safety Check

Before anything else, verify the current working directory is safe. **Hard-block** if the current directory matches any of these:

- `~/` (user home directory)
- `/` (filesystem root)
- `/etc`, `/usr`, `/var`, `/bin`, `/sbin`, `/lib`, `/opt`, `/sys`, `/proc`, `/dev`
- `/tmp`, `/private/tmp`
- Any directory containing a `go.mod` with `module` that doesn't match the user's intended module path (existing Go project)

If the current directory is suspicious (e.g., has many unrelated files, is a system directory, or is another project's root), warn the user and ask for confirmation before proceeding. Suggest creating a new subdirectory instead.

The skill runs in the **current directory** — all files are written to `./`. There is no target directory question.

### Questions

Read `skills/forge-init/data/questions.md` for the full question definitions.

Ask the user the following using `AskUserQuestion`:

1. **App name and Go module path** — If `$ARGUMENTS` is provided, use it as app name; still ask for module path (suggest `github.com/<user>/<app-name>`).
2. **Run mode** — Single-domain (`app.Run()`) or multi-domain (`forge.Run()`).
3. **Subsystems** — Multi-select from:
   - A. PostgreSQL, B. Redis, C. Sessions, D. Jobs, E. Storage
   - F. templ, G. HTMX, H. Alpine.js, I. TailwindCSS
   - J. Email, K. OAuth, L. None

After Q3, read `skills/forge-init/data/subsystem-deps.md` and resolve dependencies:

- Jobs without DB → auto-add DB, inform user
- OAuth without Sessions → recommend adding Sessions
- ANY of templ/htmx/alpine/tailwind selected → note that shared `assets/` directory and static embed will be created once
- Show final resolved subsystem list and get confirmation

---

## Phase 1: Create Directory Structure

Read `skills/forge-init/data/directory-structure.md`.

Create the base directories and any conditional directories for enabled subsystems in the current directory (`./`). Add `.gitkeep` files in empty directories.

Key rules:
- When `db` is selected, create `db/{migrations,queries}/` tree, `internal/repository/`, and generate files per `templates/db-init.md` and `templates/sqlc.md`
- When ANY frontend option (templ, htmx, alpine, tailwind) is selected, create `assets/embed.go` per `templates/assets-embed.md` and the shared `assets/{src,static/{css,js,img}}` tree — only once
- When `mailer` is selected, create `templates/emails/` and `templates/emails/layouts/` directories (email template files are generated in Phase 4)

---

## Phase 2: Generate config.go

Read `skills/forge-init/data/base-app.md` for the config.go template and the **Import Injection for Config Types** table.

For each enabled subsystem, read its file from `skills/forge-init/data/subsystems/<key>.md` and extract:

- **Imports (config.go)** section
- **Config Field** section

Compose `cmd/config.go` by inserting subsystem imports and config fields into the base template. Use the actual package-qualified config types:

| Subsystem | Config field type    | Import                                              |
|-----------|----------------------|-----------------------------------------------------|
| db        | `db.Config`          | `"github.com/dmitrymomot/forge/pkg/db"`             |
| redis     | `forgeredis.Config`  | `forgeredis "github.com/dmitrymomot/forge/pkg/redis"` |
| jobs      | `job.Config`         | `"github.com/dmitrymomot/forge/pkg/job"`            |
| storage   | `storage.Config`     | `"github.com/dmitrymomot/forge/pkg/storage"`        |

Remove placeholder comments from the final output.

---

## Phase 3: Generate main.go

Read `skills/forge-init/data/base-app.md` and select the correct main.go template based on the user's run mode choice (single-domain or multi-domain).

For each enabled subsystem, read its file from `skills/forge-init/data/subsystems/<key>.md` and extract:

- **Imports (main.go)** section
- **Init Code** section
- **Health Check** section
- **App Option** section
- **Shutdown Hook** section

### Shared Frontend Concern

When ANY of `templ`, `htmx`, `alpine`, or `tailwind` is selected, add **once** (not per option):

1. Import `"{{MODULE_PATH}}/assets"` in main.go
2. App option: `forge.WithStaticFiles("/static/", assets.StaticFS, "static")`

No `"embed"` import or `//go:embed` directive in main.go — the embed lives in `assets/embed.go`.

### DB Embed Concern

When `db` is selected, add:

1. Import `dbmigrations "{{MODULE_PATH}}/db/migrations"` in main.go
2. Use `dbmigrations.FS` in init code: `db.WithMigrations(dbmigrations.FS)`

No `"embed"` import or `//go:embed` directive in main.go — the embed lives in `db/migrations/embed.go`.

### Assembly

Compose `cmd/main.go` by:

1. Merging all imports (deduplicate) — embed-based imports are regular package imports, no `"embed"` needed
2. Placing init code in dependency order (db first, then redis, storage, mailer, oauth)
3. Assembling health checks, app options, and shutdown hooks
4. Replacing `{{APP_NAME}}` with the actual app name
5. Removing unused placeholder comments and the `_ = ctx` line if ctx is used

---

## Phase 4: Generate Support Files

Read each template file and generate the corresponding project file:

- Read `skills/forge-init/data/templates/air-toml.md` → write `.air.toml`
  - If `templ` is selected, use the extended `include_ext` with `"templ"` added
- Read `skills/forge-init/data/templates/taskfile.md` → write `Taskfile.yml` (include conditional sections based on enabled subsystems)
  - If `db` is selected, include the `db:migration:create` and `db:generate` tasks
  - If any of htmx/alpine/tailwind is selected, include the `assets:download` task with only the curl lines for selected libraries
- Read `skills/forge-init/data/templates/docker-compose.md` → write `docker-compose.yml` (assemble services from enabled subsystems' Docker Service sections)
- Read `skills/forge-init/data/templates/gitignore.md` → write `.gitignore`
- If `mailer` is selected, read the Generated Files section from `skills/forge-init/data/subsystems/mailer.md` and create `templates/emails/embed.go`, `templates/emails/welcome.md`, and `templates/emails/layouts/base.html`. Replace `{{APP_NAME}}` with the actual app name. Preserve Go template syntax (`{{.Content}}`, `{{.Metadata.Subject}}`, `{{.Name}}`, etc.) verbatim — only replace `{{APP_NAME}}` and `{{MODULE_PATH}}` scaffold placeholders.

---

## Phase 5: Generate Environment Files

Read `skills/forge-init/data/templates/env-vars.md` for the assembly rules.

For each enabled subsystem, use the "Env Vars (dev)" and "Env Vars (example)" sections from its `data/subsystems/<key>.md` file.

Generate two files:

- `.env` — dev-ready values matching docker-compose credentials/ports
- `.env.example` — same keys with placeholder values

Replace `{{APP_NAME}}` with the actual app name in all values.

---

## Phase 6: Initialize Go Module

Run these commands in the current directory:

```bash
go mod init <module-path>
```

Then edit `go.mod` to set `go 1.25` and add the tool directive. Read `skills/forge-init/data/base-app.md` for the full tool list.

The tool directive block to add:

```
tool (
	github.com/air-verse/air
	github.com/golangci/golangci-lint/v2/cmd/golangci-lint
	github.com/pressly/goose/v3/cmd/goose
	golang.org/x/tools/cmd/goimports
)
```

If `templ` is selected, also add to the tool block:

```
	github.com/a-h/templ/cmd/templ
	github.com/templui/templui/cmd/templui
```

If `db` is selected, also add to the tool block:

```
	github.com/sqlc-dev/sqlc/cmd/sqlc
```

Then run:

```bash
go get github.com/dmitrymomot/forge@latest
go mod tidy
```

If any of htmx/alpine/tailwind is selected, also run:

```bash
task assets:download
```

---

## Phase 7: Verify Compilation

Run:

```bash
go build -o /dev/null ./...
```

If compilation fails:

1. Read the error output carefully
2. Fix the issue in the generated code
3. Run `go mod tidy` again
4. Re-run `go build -o /dev/null ./...`
5. Repeat until compilation succeeds

---

## Phase 8: Summary

Display a summary to the user:

1. **Project created** in the current directory
2. **Module**: `<module-path>`
3. **Run mode**: single-domain / multi-domain
4. **Enabled subsystems**: list with checkmarks
5. **Generated files**: tree listing
6. **Next steps**:
    - `task docker:up` (if Docker services exist)
    - `task assets:download` (if htmx/alpine/tailwind selected — remind them to re-run after updates)
    - `task dev` to start developing
    - Create handlers in `internal/handler/`
    - Create migrations with `task db:migration:create -- <name>` (if db enabled)
    - Generate repository code with `task db:generate` after adding SQL queries (if db enabled)
    - Implement session store (if sessions enabled)
    - Customize email templates in `templates/emails/` (if mailer enabled)
    - Set up OAuth credentials (if oauth enabled)
    - Health checks available at `/_live` and `/_ready`

If `templ` is enabled, add to next steps:

- **templ docs**: https://templ.guide/
- **templui docs**: https://templui.io/docs/how-to-use
- Run `templ generate` to compile `.templ` files to Go code
