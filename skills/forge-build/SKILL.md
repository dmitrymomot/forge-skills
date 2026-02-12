---
name: forge-build
description: "Build complete features for Forge apps: handlers, migrations, jobs, auth, email, storage, SSE. Triggers on: add feature, create handler, new table, migration, endpoint, API, CRUD, login, register, job, worker, upload, email, SSE."
argument-hint: <feature description>
context: fork
allowed-tools: Read, Glob, Grep, Edit, Write, Bash
---

# Forge Feature Builder

You build complete features for Forge framework applications. You generate migrations, queries, handlers, tasks, email templates, views, and wire everything into `cmd/main.go`. Follow the phases below precisely.

The feature description comes from `$ARGUMENTS`.

---

## Phase 0: Discover (silent — no output to user)

Gather project context before doing anything:

1. **Read `go.mod`** — extract module path. If missing → hard stop: _"No go.mod found. Run `/forge-init` to scaffold a new Forge app first."_
2. **Detect enabled subsystems** by grepping `cmd/config.go` for config types:
   - `db.Config` → database enabled
   - `job.Config` → background jobs enabled
   - `storage.Config` → file storage enabled
   - `mailer.Config` or `smtp.Config` → email enabled
   - OAuth config types → OAuth enabled
   - `forgeredis.Config` → Redis enabled
   - Grep `cmd/main.go` for `forge.WithSession` → sessions enabled
3. **If DB enabled:**
   - List `db/migrations/*.sql` → find highest migration number (next = highest + 1, zero-padded to 5 digits). If none exist, start at `00001`
   - List `db/queries/*.sql` → inventory existing query files
   - Grep existing migrations for `CREATE TABLE` → build awareness of existing tables, columns, FK targets
4. **Check frontend state:** detect `.templui.json` existence
5. **List existing files:**
   - `internal/handler/*.go` — existing handlers
   - `internal/task/*.go` — existing tasks
6. **Always read:** `skills/forge-build/data/conventions.md`

---

## Phase 1: Classify

Analyze `$ARGUMENTS` to determine which categories this feature touches. Classification is **additive** — all matching categories fire simultaneously.

| Signal Keywords | Category | Load Data File |
|---|---|---|
| table, migration, column, schema, database, alter, index | DB | `data/db.md` |
| handler, endpoint, route, API, CRUD, REST, resource | Handler | `data/handler.md` |
| login, register, auth, session, password, OAuth, RBAC | Auth | `data/auth.md` |
| job, worker, background, async, queue, task, process | Task | `data/task.md` |
| scheduled, cron, periodic, recurring, daily, hourly | Scheduled | `data/scheduled-task.md` |
| email, mail, notification, invite, welcome | Email | `data/email.md` |
| upload, file, storage, download, image, attachment | Storage | `data/storage.md` |
| page, view, template, render, layout, partial | Templ | `data/templ.md` |
| htmx, swap, trigger, hx-*, boost, partial update | HTMX | `data/htmx.md` |
| SSE, real-time, live, stream, events, push | SSE | `data/sse.md` |

**Implicit rules:**
- Entity name (e.g., "blog posts", "products") → DB + Handler automatically
- Auth features → Auth + DB (users table if missing) + Handler
- Email features → Email + Task (if jobs enabled, send async)
- If ambiguous → ask user before proceeding

**Pre-flight check:** For each classified category, verify the required subsystem is enabled:
- DB → `db.Config` must be present
- Task/Scheduled → `job.Config` must be present
- Storage → `storage.Config` must be present
- Email → `mailer.Config` must be present
- Auth → sessions must be enabled

If a required subsystem is missing → warn the user and tell them what to add to `cmd/config.go` and `cmd/main.go` manually. Do NOT add subsystems yourself.

---

## Phase 2: Plan

For each classified category, build specific details:

### DB
- Table name (pluralized snake_case)
- Columns with types (inferred from `data/db.md` type mappings)
- Auto-include: `id VARCHAR(26) PRIMARY KEY`, `created_at`, `updated_at`, `updated_at` trigger
- Detect patterns: soft delete, enum, FK, junction table
- Migration SQL (Up + Down)
- Proposed queries: always GetByID, List, Create, Update, Delete; plus GetBy<Unique>, ListBy<FK>, Search, soft delete variants as appropriate
- If existing query file: parse existing names, propose only missing queries

### Handler
- Handler struct name, file path
- Dependencies (repo, mailer, storage, etc.)
- Routes (method + path)
- Request structs with binding tags
- Whether JSON API or HTML rendering

### Task
- Worker struct, args struct, file path
- Task name (snake_case)
- Enqueue locations (which handlers call it)
- Retry config if non-default

### Scheduled
- Task struct, file path
- Schedule expression
- Dependencies

### Auth
- Which routes (login, register, logout, OAuth)
- Users table migration if missing
- Password hashing approach

### Email
- Template file path and variables
- Sync vs async sending
- Layout reference

### Storage
- Upload handler pattern
- DB columns for file keys
- Validation rules

### Templ
- Page and partial template files
- Layout usage
- View models if needed

### HTMX
- Swap targets and modes
- Trigger patterns
- Response headers

### SSE
- Event channel and producer
- Event types
- Client-side integration

### Wiring
- `cmd/main.go` changes: new imports + handler/task registrations

---

## Phase 3: Ask (NEVER skip this phase)

Present the full plan with clear formatting:

### Migration SQL Preview (if DB)
Show complete Up and Down SQL.

### Route Table (if handler)
| Method | Path | Handler | Auth? |
|---|---|---|---|

### Query List (if DB)
| Query Name | Annotation | Description |
|---|---|---|

### Worker Definition (if tasks)
| Task Name | Args | Trigger |
|---|---|---|

### All Target File Paths
List every file that will be created or modified.

### Main.go Wiring Changes
Show what imports and registrations will be added.

**Get user confirmation before generating anything.** Let the user adjust columns, routes, queries, or naming.

---

## Phase 4: Generate

Execute in strict dependency order:

### Step 1: Migration files (if DB)
Write `db/migrations/NNNNN_<name>.sql` with goose format:
```sql
-- +goose Up
<SQL>

-- +goose Down
<reverse SQL>
```

### Step 2: Query files (if DB)
- New table → write `db/queries/<table>.sql`
- Existing table → use `Edit` tool to append new queries (never overwrite existing)

### Step 3: sqlc generate (if DB)
```bash
go tool sqlc generate -f db/sqlc.yml
```

### Step 4: Email templates (if email)
Write `templates/emails/<name>.md` with YAML frontmatter.

### Step 5: Task/scheduled files (if tasks)
Write `internal/task/<name>.go`

### Step 6: Handler files
Write `internal/handler/<name>.go`

### Step 7: Templ views (if rendering needed)
Write `.templ` files with plain HTML (no styled components — that's `/templui`'s job).

### Step 8: Wire cmd/main.go
Use `Edit` tool to add:
- New imports
- Handler registrations in `forge.WithHandlers()`
- Task registrations in `forge.WithJobs()` or `forge.WithScheduledTask()`

---

## Phase 5: Verify

Run in order, fix and retry until clean:

1. `go tool sqlc generate -f db/sqlc.yml` (if DB — re-verify after any fixes)
2. `go tool templ generate` (if templ files were created)
3. `go build -o /dev/null ./...`

If any step fails:
1. Read the error output
2. Fix the source file
3. Re-run the failing step
4. Repeat until all pass

---

## Phase 6: Summary

### Files Created/Modified
List each file path with `(created)` or `(modified)` label.

### Generated Query Functions (if DB)
| Function | Signature | Description |
|---|---|---|

### Available Routes (if handler)
| Method | Path | Description |
|---|---|---|

### Registered Tasks (if tasks)
| Task Name | Schedule | Description |
|---|---|---|

### Next Steps (contextual)
- "Use `/templui` to generate polished UI for the views" (if templ stubs created)
- "Run `/go-test` to generate tests" (always)
- "Customize email template at `templates/emails/<name>.md`" (if email)
- "Migrations auto-run on app start via `db.Open()` — no manual step needed in dev"

---

## Explicit Exclusions

**Never do any of these — they belong to other skills or manual steps:**

- Never scaffold new projects → `/forge-init`
- Never generate polished/styled UI → `/templui`
- Never add subsystems to project infrastructure (docker-compose, .env, config structs)
- Never generate tests → `/go-test`
- Never generate docs or commits → `/go-doc`, `/git-commit`
- Never run `go get` to add dependencies
- Never install templui components
- Never modify docker-compose.yml or .env files

---

## Important Rules

- **Forge IDs**: Always `VARCHAR(26)` — never UUID, SERIAL, IDENTITY
- **Timestamps**: Always `TIMESTAMPTZ NOT NULL DEFAULT now()` for created_at/updated_at
- **Trigger**: Every new table gets an `updated_at` trigger
- **Naming**: SQL snake_case, Go PascalCase, URLs kebab-case, JSON camelCase
- **Tags**: Validation/sanitization use semicolons: `validate:"required;max:100"`
- **Build**: `go build -o /dev/null ./...` — never leave binaries
- **Edit, don't overwrite**: When adding to existing files, use `Edit` to append/insert
- **Session safety**: Extract `c.UserID()` etc. before goroutines — NOT goroutine-safe
- **Modern Go**: `for range n`, `min()`/`max()`, `slices.Contains()`, no `v := v`
