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
      - go tool air

  build:
    desc: Build the application binary
    cmds:
      - go build -o ./tmp/main ./cmd/...

  run:
    desc: Run the built binary
    deps: [build]
    cmds:
      - ./tmp/main

  fmt:
    desc: Format code and imports
    cmds:
      - go fmt ./...
      - go tool goimports -w -local {{MODULE_PATH}} .
      - go tool betteralign -apply ./...

  test:
    desc: Run all tests with race detection and coverage
    cmds:
      - go clean -testcache && go test -race -cover ./...

  lint:
    desc: Run code linters
    cmds:
      - go vet ./...
      - go build -o /dev/null ./...
      - go tool golangci-lint run ./...
      - go tool nilaway ./...
      - go tool betteralign ./...
      - go tool modernize $(go list ./...)

  deps:
    desc: Update and tidy go module dependencies
    cmds:
      - go get -u ./...
      - go mod tidy
```

## Conditional: DB tasks (when `db` is enabled)

```yaml
db:migration:create:
  desc: Create a new SQL migration file
  cmds:
    - go tool goose -dir db/migrations create {{.CLI_ARGS}} sql

db:generate:
  desc: Generate Go code from SQL queries
  cmds:
    - go tool sqlc generate -f db/sqlc.yml
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
    - curl -sL https://unpkg.com/htmx.org/dist/ext/sse.js -o assets/static/js/htmx-sse.js
    - curl -sL https://unpkg.com/htmx.org/dist/ext/ws.js -o assets/static/js/htmx-ws.js
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

## Conditional: Integration test tasks (when any Docker service exists — db, redis, storage, or mailer)

```yaml
test:up:
  desc: Start test infrastructure (docker containers)
  cmds:
    - docker compose -f docker-compose.test.yml up -d --wait {{SERVICES_LIST}}
    # If storage is enabled, add this second line:
    # - docker compose -f docker-compose.test.yml up storage-bucket-init

test:down:
  desc: Stop and remove test infrastructure
  cmds:
    - docker compose -f docker-compose.test.yml down -v --remove-orphans

test:integration:
  desc: Run integration tests with docker infrastructure
  deps: [test:up]
  cmds:
    - defer: { task: test:down }
    - go test -tags=integration -count=1 -race -cover $(grep -rl '//go:build integration' . --include='*_test.go' | xargs -I{} dirname {} | sort -u)
```

Assembly notes:
- `{{SERVICES_LIST}}` is built from the enabled subsystems' Docker service names (e.g., `postgres redis storage mailpit`).
- If storage is enabled, add the `storage-bucket-init` line to `test:up` after the main `--wait` command.
- Only include this section when at least one Docker service exists (same condition as docker tasks).

## Conditional: templ generate task (when `templ` is enabled)

```yaml
templ:generate:
  desc: Generate Go code from .templ files
  cmds:
    - go tool templ generate
```

## Conditional: Setup task

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

- Run via `go tool task <taskname>` — Task is included in the go.mod tool directive.
- The `dotenv` directive loads `.env` file automatically.
- The `fmt` task formats code with `go fmt`, organizes imports with `goimports` (using the module path for local import grouping), and aligns struct fields with `betteralign`.
- The `lint` task runs a comprehensive linting pipeline matching the Forge framework's own standards: `go vet`, build check, `golangci-lint`, `nilaway`, `betteralign`, and `modernize`.
- The `test` task runs tests with race detection and coverage reporting.
- The `test:integration` task starts Docker test infrastructure, runs integration tests (files tagged with `//go:build integration`), and tears down infrastructure automatically via `defer`.
- The `db:migration:create` task uses goose to create migration files in the `db/migrations/` directory. Usage: `go tool task db:migration:create -- add_users_table`
- The `db:generate` task runs sqlc to generate Go code from SQL queries into `internal/repository/`. Usage: `go tool task db:generate`
- `{{MODULE_PATH}}` in the `fmt` task must be replaced with the actual Go module path during generation.
- `{{SERVICES_LIST}}` in the `test:up` task must be replaced with the space-separated list of Docker service names for the enabled subsystems (e.g., `postgres redis storage mailpit`).
- Add custom tasks as needed for the specific project.
