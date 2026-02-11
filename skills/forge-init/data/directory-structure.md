# Project Directory Structure

## Base (always created)

```
./
├── cmd/
│   ├── main.go
│   └── config.go
├── internal/
│   └── handler/       # HTTP handlers (route groups)
├── .air.toml
├── .env
├── .env.example
├── .gitignore
├── docker-compose.yml
├── Taskfile.yml
├── go.mod
└── go.sum
```

## Conditional Directories

| Directory                      | Created when                     | Purpose                              |
|--------------------------------|----------------------------------|--------------------------------------|
| `internal/worker/`             | jobs                             | Background job workers               |
| `db/migrations/`               | db                               | SQL migration files (goose) + embed  |
| `db/queries/`                  | db                               | sqlc query files                     |
| `internal/repository/`         | db                               | sqlc generated Go code               |
| `assets/embed.go`              | templ OR htmx OR alpine OR tailwind | Embed package for static files    |
| `assets/src/`                  | templ OR htmx OR alpine OR tailwind | Source assets (pre-build)          |
| `assets/static/css/`           | templ OR htmx OR alpine OR tailwind | Compiled/vendored CSS              |
| `assets/static/js/`            | templ OR htmx OR alpine OR tailwind | Compiled/vendored JS               |
| `assets/static/img/`           | templ OR htmx OR alpine OR tailwind | Static images                      |
| `templates/emails/`            | mailer                           | Email templates                      |
| `templates/emails/layouts/`    | mailer                           | Email HTML layouts                   |

## DB Directory Structure (when `db` is selected)

```
db/
├── sqlc.yml              # sqlc configuration (generated)
├── migrations/
│   ├── embed.go          # Embed package exposing FS
│   └── 00001_init.sql    # Initial migration
└── queries/
    └── .gitkeep
internal/repository/
    └── .gitkeep
```

- `db/sqlc.yml` — sqlc configuration, see `templates/sqlc.md`
- `db/migrations/embed.go` — embed package, see `templates/db-init.md`
- `db/migrations/00001_init.sql` — initial migration with `update_updated_at_column()` trigger, see `templates/db-init.md`
- `internal/repository/` — sqlc output directory, initially empty with `.gitkeep`

## Directory Creation Rules

1. Always create `cmd/`, `internal/handler/`
2. Create conditional directories based on selected subsystems
3. Add a `.gitkeep` file in empty directories so they are tracked by git
4. When `db` is selected, create the full `db/{migrations,queries}/` tree and `internal/repository/` with generated files per `templates/db-init.md` and `templates/sqlc.md`
5. When ANY frontend option (templ, htmx, alpine, tailwind) is selected, create `assets/embed.go` per `templates/assets-embed.md` and the full `assets/{src,static/{css,js,img}}` tree — only once, not per option
6. When `mailer` is selected, create `templates/emails/` and `templates/emails/layouts/`, then generate `templates/emails/embed.go`, `templates/emails/welcome.md`, and `templates/emails/layouts/base.html` from the mailer subsystem's Generated Files section. Do NOT add `.gitkeep` in these directories since they contain actual files.
