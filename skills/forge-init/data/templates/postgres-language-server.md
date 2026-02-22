# postgres-language-server.jsonc

Generate this file at the project root when `db` is selected.

## Content

```jsonc
{
  // PostgreSQL Language Server configuration
  // Silences false positives for DROP statements in goose migration Down blocks

  "$schema": "https://pgtools.dev/schema.json",

  "migrations": {
    // Allow destructive operations in migration files
    "allowDestructiveStatements": true
  }
}
```

## Notes

- The PostgreSQL language server linter flags `DROP TABLE`, `DROP COLUMN`, and `DROP DATABASE` as safety violations by default.
- These statements are expected and correct in goose `-- +goose Down` migration blocks.
- This config silences those false positives so the linter remains useful without noisy warnings.
