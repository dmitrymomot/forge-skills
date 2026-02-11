# sqlc.yml — SQL Code Generator Configuration

Generate this file at `db/sqlc.yml` when `db` is selected.

## Template

```yaml
version: "2"
sql:
  - schema: "migrations"
    queries: "queries"
    engine: "postgresql"
    gen:
      go:
        package: "repository"
        sql_package: "pgx/v5"
        out: "../internal/repository"
        emit_json_tags: true
        emit_prepared_queries: false
        emit_interface: true
        emit_empty_slices: true
        emit_all_enum_values: true
        emit_exported_queries: true
        overrides:
          - db_type: "pg_catalog.timestamp"
            go_type:
              import: "time"
              type: "Time"
            nullable: false
          - db_type: "pg_catalog.timestamp"
            go_type:
              import: "time"
              type: "Time"
              pointer: true
            nullable: true
          - db_type: "pg_catalog.varchar"
            go_type:
              type: "string"
              pointer: true
            nullable: true
          - db_type: "text"
            go_type:
              type: "string"
              pointer: true
            nullable: true
          - db_type: "timestamptz"
            go_type:
              import: "time"
              type: "Time"
            nullable: false
          - db_type: "timestamptz"
            go_type:
              import: "time"
              type: "Time"
              pointer: true
            nullable: true
```

## Notes

- **No UUID overrides** — Forge uses `pkg/id` to generate string-based IDs in Go. Columns use `VARCHAR` or `TEXT` types, which map to `string` natively.
- Paths are relative to `db/sqlc.yml`: `schema: "migrations"` resolves to `db/migrations/`, `queries: "queries"` resolves to `db/queries/`, and `out: "../internal/repository"` resolves to `internal/repository/`.
- Run with: `go tool sqlc generate -f db/sqlc.yml`
