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
| `migrations/`                  | db                               | SQL migration files (goose)          |
| `assets/src/`                  | templ OR htmx OR alpine OR tailwind | Source assets (pre-build)          |
| `assets/static/css/`           | templ OR htmx OR alpine OR tailwind | Compiled/vendored CSS              |
| `assets/static/js/`            | templ OR htmx OR alpine OR tailwind | Compiled/vendored JS               |
| `assets/static/img/`           | templ OR htmx OR alpine OR tailwind | Static images                      |
| `templates/pages/`             | templ                            | Page templates (.templ files)        |
| `templates/partials/`          | templ                            | Partial/fragment templates           |
| `templates/layouts/`           | templ                            | Layout templates                     |
| `templates/emails/`            | mailer                           | Email templates                      |

## Directory Creation Rules

1. Always create `cmd/`, `internal/handler/`
2. Create conditional directories based on selected subsystems
3. Add a `.gitkeep` file in empty directories so they are tracked by git
4. When `db` is selected, create `migrations/` with a `.gitkeep` file
5. When ANY frontend option (templ, htmx, alpine, tailwind) is selected, create the full `assets/{src,static/{css,js,img}}` tree — only once, not per option
6. When `templ` is selected, create `templates/{pages,partials,layouts}/` with `.gitkeep` files
