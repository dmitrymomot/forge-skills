# Taskfile.yml — Task Runner Configuration

Generate this file at the project root for common development tasks using [Task](https://taskfile.dev).

## Base Template (always included)

```yaml
version: "3"

dotenv: [".env"]

tasks:
  dev:
    desc: Run the application with hot reload
    cmds:
      - air

  build:
    desc: Build the application binary
    cmds:
      - go build -o ./tmp/main ./cmd/...

  run:
    desc: Run the built binary
    deps: [build]
    cmds:
      - ./tmp/main

  test:
    desc: Run tests
    cmds:
      - go test ./...

  lint:
    desc: Run linter
    cmds:
      - golangci-lint run ./...

  tidy:
    desc: Tidy go modules
    cmds:
      - go mod tidy
```

## Conditional: DB tasks (when `db` is enabled)

```yaml
  db:migration:create:
    desc: Create a new SQL migration file
    cmds:
      - goose -dir migrations create {{.CLI_ARGS}} sql
```

## Conditional: Assets download task (when ANY of `htmx`, `alpine`, or `tailwind` is enabled)

Build the `assets:download` task by including only the curl commands for the selected libraries:

```yaml
  assets:download:
    desc: Download vendored frontend assets
    cmds:
      # Include the lines below based on which frontend libraries are selected:
      # htmx:
      - curl -sL https://unpkg.com/htmx.org/dist/htmx.min.js -o assets/static/js/htmx.min.js
      # alpine:
      - curl -sL https://unpkg.com/alpinejs/dist/cdn.min.js -o assets/static/js/alpine.min.js
      # tailwind:
      - curl -sL https://unpkg.com/@tailwindcss/browser@4/dist/cdn.min.js -o assets/static/js/tailwind.min.js
```

Only include the curl lines for selected libraries. If none of htmx/alpine/tailwind are selected, omit the entire `assets:download` task.

## Conditional: Docker tasks (when any Docker service exists)

```yaml
  docker:up:
    desc: Start Docker services
    cmds:
      - docker compose up -d

  docker:down:
    desc: Stop Docker services
    cmds:
      - docker compose down

  docker:logs:
    desc: Tail Docker service logs
    cmds:
      - docker compose logs -f
```

## Conditional: Setup task (always included)

```yaml
  setup:
    desc: Initial project setup
    cmds:
      - go mod tidy
      - task: docker:up
      - task: assets:download
      - echo "Project setup complete. Run 'task dev' to start developing."
```

Adjust the setup task:
- If no Docker services exist, remove the `task: docker:up` line.
- If no frontend download libraries are selected (htmx/alpine/tailwind), remove the `task: assets:download` line.

## Notes

- Task must be installed separately: https://taskfile.dev/installation/
- The `dotenv` directive loads `.env` file automatically.
- The `db:migration:create` task uses goose to create migration files in the `migrations/` directory. Usage: `task db:migration:create -- add_users_table`
- Add custom tasks as needed for the specific project.
