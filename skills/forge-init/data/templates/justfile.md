# justfile — Task Runner Configuration

Generate this file at the project root for common development tasks using [just](https://github.com/casey/just).

## Placeholder Syntax Note

This template uses `{{ }}` for two distinct purposes:

- **ALL_CAPS placeholders** like `{{MODULE_PATH}}` and `{{SERVICES_LIST}}` are **skill placeholders** — replaced at generation time with actual values. They do NOT appear in the final file.
- **lowercase with spaces** like `{{ path }}` and `{{ name }}` are **justfile native interpolation** — preserved verbatim in the generated file. These are justfile's own variable/parameter syntax.

## Base Template (always included)

```just
set dotenv-load

# List available recipes
default:
    @just --list

[group('dev')]
dev:
    go tool air

[group('dev')]
build:
    go build -o ./tmp/main ./cmd/...

[group('dev')]
run: build
    ./tmp/main

[group('quality')]
fmt:
    go fmt ./...
    go tool goimports -w -local {{MODULE_PATH}} .
    go tool betteralign -apply ./...

[group('quality')]
test:
    go clean -testcache && go test -race -cover ./...

[group('quality')]
lint:
    go vet ./...
    go build -o /dev/null ./...
    go tool golangci-lint run ./...
    go tool nilaway ./...
    go tool betteralign ./...
    go tool modernize $(go list ./...)

[group('quality')]
check: fmt lint test

[group('dev')]
clean:
    rm -rf ./tmp/
    go clean -testcache
    go clean -cache

[group('dev')]
deps:
    go get -u ./...
    go mod tidy
```

## Conditional: DB recipes (when `db` is enabled)

```just
[group('db')]
db-migration-create name:
    go tool goose -dir db/migrations create {{ name }} sql

[group('codegen')]
db-generate:
    go tool sqlc generate -f db/sqlc.yml

[group('db')]
db-migrate:
    go tool goose -dir db/migrations postgres "$DATABASE_URL" up

[group('db')]
db-migrate-down:
    go tool goose -dir db/migrations postgres "$DATABASE_URL" down

[group('db')]
db-status:
    go tool goose -dir db/migrations postgres "$DATABASE_URL" status

[group('db')]
[confirm("This will destroy all data. Continue?")]
db-reset:
    go tool goose -dir db/migrations postgres "$DATABASE_URL" down-to 0
    go tool goose -dir db/migrations postgres "$DATABASE_URL" up
```

## Conditional: Assets download recipe (when ANY of `htmx`, `alpine`, or `tailwind` is enabled)

Build the `assets-download` recipe by including only the curl commands for the selected libraries:

```just
[group('assets')]
assets-download:
    # Include the lines below based on which frontend libraries are selected:
    # htmx:
    curl -sL https://unpkg.com/htmx.org/dist/htmx.min.js -o assets/static/js/htmx.min.js
    curl -sL https://unpkg.com/htmx.org/dist/ext/sse.js -o assets/static/js/htmx-sse.js
    curl -sL https://unpkg.com/htmx.org/dist/ext/ws.js -o assets/static/js/htmx-ws.js
    # alpine:
    curl -sL https://unpkg.com/alpinejs/dist/cdn.min.js -o assets/static/js/alpine.min.js
    # tailwind (elements.js — only when templ is NOT selected):
    curl -sL "https://cdn.jsdelivr.net/npm/@tailwindplus/elements@1" | sed '/^\/\/# sourceMappingURL=/d' > "assets/static/js/elements.min.js"
```

Only include the curl lines for selected libraries. If none of htmx/alpine/tailwind are selected, omit the entire `assets-download` recipe.

## Conditional: CSS build recipe (when `tailwind` is enabled)

```just
[group('assets')]
css:
    # Single-domain (one CSS file):
    tailwindcss -i assets/src/app.css -o assets/static/css/app.css --minify
    # Multi-domain (one line per domain — include only the relevant lines):
    # tailwindcss -i assets/src/website.css -o assets/static/css/website.css --minify
    # tailwindcss -i assets/src/app.css -o assets/static/css/app.css --minify
```

For single-domain projects, include only the single `app.css` line. For multi-domain projects, include one line per domain CSS source file (uncomment and adjust as needed). Omit this recipe entirely if `tailwind` is not selected.

## Conditional: Docker recipes (when any Docker service exists)

```just
[group('docker')]
docker-up:
    docker compose up -d

[group('docker')]
docker-down:
    docker compose down

[group('docker')]
docker-logs:
    docker compose logs -f
```

## Conditional: Integration test recipes (when any Docker service exists — db, redis, storage, or mailer)

```just
alias ti := test-integration

[group('test')]
test-integration:
    #!/usr/bin/env bash
    set -euo pipefail
    docker compose -f docker-compose.test.yml up -d --wait {{SERVICES_LIST}}
    # If storage is enabled, add this line:
    # docker compose -f docker-compose.test.yml up storage-bucket-init
    trap 'docker compose -f docker-compose.test.yml down -v --remove-orphans' EXIT
    go test -tags=integration -count=1 -race -cover $(grep -rl '//go:build integration' . --include='*_test.go' | xargs -I{} dirname {} | sort -u)
```

Assembly notes:
- `{{SERVICES_LIST}}` is built from the enabled subsystems' Docker service names (e.g., `postgres redis storage mailpit`).
- If storage is enabled, add the `storage-bucket-init` line after the main `--wait` command.
- Only include this section when at least one Docker service exists (same condition as docker recipes).
- The recipe is fully self-contained: starts test infrastructure, runs tests, and tears down via `trap` on exit.
- `just ti` is a shorthand alias for `just test-integration`.

## Conditional: templ generate recipe (when `templ` is enabled)

```just
[group('codegen')]
templ-generate:
    go tool templ generate
```

## Conditional: generate meta-recipe (when `db` OR `templ` is enabled)

```just
[group('codegen')]
generate:
    # Include lines based on enabled code generators:
    # db: just db-generate
    # templ: just templ-generate
```

Assembly: include only the `just <x>` lines for enabled subsystems. If only one generator is enabled, the recipe still exists (it's the canonical entry point for code generation). Omit the entire recipe if neither `db` nor `templ` is enabled.

## Conditional: Setup recipe

```just
[group('dev')]
setup:
    go mod tidy
    just docker-up
    just assets-download
    echo "Project setup complete. Run 'just dev' to start developing."
```

Adjust the setup recipe:

- If no Docker services exist, remove the `just docker-up` line.
- If no frontend download libraries are selected (htmx/alpine/tailwind), remove the `just assets-download` line.
- If `tailwind` is enabled, add `just css` after the `just assets-download` line to compile CSS after downloading assets.

## Notes

- Run via `just <recipe>` — `just` is a system binary installed via `brew install just` (macOS) or `cargo install just`.
- The `default` recipe runs `just --list`, showing all recipes organized by group.
- The `set dotenv-load` directive loads `.env` file automatically.
- Recipes are organized into groups: `dev`, `quality`, `db`, `codegen`, `assets`, `docker`, `test`. Groups appear in `just --list` output for discoverability.
- The `fmt` recipe formats code with `go fmt`, organizes imports with `goimports` (using the module path for local import grouping), and aligns struct fields with `betteralign`.
- The `lint` recipe runs a comprehensive linting pipeline matching the Forge framework's own standards: `go vet`, build check, `golangci-lint`, `nilaway`, `betteralign`, and `modernize`.
- The `check` recipe runs `fmt`, `lint`, and `test` in sequence — use as a pre-commit gate.
- The `test` recipe runs tests with race detection and coverage reporting.
- The `clean` recipe removes build artifacts (`./tmp/`) and Go caches.
- The `test-integration` recipe starts Docker test infrastructure, runs integration tests (files tagged with `//go:build integration`), and tears down infrastructure automatically via `trap`. Alias: `just ti`.
- The `generate` recipe runs all enabled code generators (sqlc, templ) in one command.
- The `db-migration-create` recipe uses goose to create migration files in the `db/migrations/` directory. Usage: `just db-migration-create add_users_table`
- The `db-generate` recipe runs sqlc to generate Go code from SQL queries into `internal/repository/`. Usage: `just db-generate`
- The `db-migrate` recipe runs pending migrations against `$DATABASE_URL`. The `db-migrate-down` recipe rolls back the last migration. The `db-status` recipe shows current migration state. The `db-reset` recipe drops all tables and re-migrates (with confirmation prompt).
- `{{MODULE_PATH}}` in the `fmt` recipe must be replaced with the actual Go module path during generation.
- `{{SERVICES_LIST}}` in the `test-integration` recipe must be replaced with the space-separated list of Docker service names for the enabled subsystems (e.g., `postgres redis storage mailpit`).
- Recipe indentation uses 4 spaces per the `.editorconfig` setting.
- Add custom recipes as needed for the specific project.
