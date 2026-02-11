# Database Init Templates

These files are generated when `db` is selected.

---

## db/migrations/embed.go

```go
package migrations

import "embed"

//go:embed *.sql
var FS embed.FS
```

This embed package exposes the migration SQL files as `embed.FS`. Import it from main.go as:

```go
dbmigrations "{{MODULE_PATH}}/db/migrations"
```

Then pass `dbmigrations.FS` to the migrator.

---

## db/migrations/00001_init.sql

```sql
-- +goose Up
-- +goose StatementBegin

-- Create a function to automatically update the updated_at column
CREATE OR REPLACE FUNCTION update_updated_at_column()
    RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = CURRENT_TIMESTAMP;
    RETURN NEW;
END;
$$ LANGUAGE 'plpgsql';

-- +goose StatementEnd

-- +goose Down
-- +goose StatementBegin

-- Drop function
DROP FUNCTION IF EXISTS update_updated_at_column();

-- +goose StatementEnd
```

### Notes

- **No `uuid-ossp` extension** — Forge generates IDs in Go via `pkg/id`, so database-level UUID generation is not needed. Columns use `VARCHAR` or `TEXT` types.
- The `update_updated_at_column()` trigger function is useful for any table with an `updated_at` column. Apply it per table in subsequent migrations:
  ```sql
  CREATE TRIGGER set_updated_at
      BEFORE UPDATE ON my_table
      FOR EACH ROW
      EXECUTE FUNCTION update_updated_at_column();
  ```
