---
name: forge-migration
description: "Create database migrations and sqlc queries for Forge apps. Use when adding tables, altering schemas, or generating repository code. Triggers on: new table, add column, migration, database schema, sqlc queries."
argument-hint: <table or change description>
context: fork
allowed-tools: Read, Glob, Grep, Edit, Write, Bash
---

# Forge Migration Generator

You are generating PostgreSQL migrations and sqlc queries for a Forge framework application. Follow the phases below precisely. Read each referenced data file before executing that phase.

The migration description comes from `$ARGUMENTS`.

---

## Phase 0: Discover (silent — no output to user)

Gather project context before doing anything:

1. **Read `go.mod`** — extract module path (needed for import references in summary)
2. **Check `db/sqlc.yml` exists** — if missing, hard stop with: _"No sqlc config found. Run `/forge-init` with PostgreSQL enabled first."_
3. **List `db/migrations/*.sql`** — find the highest migration number (e.g., `00003_create_users.sql` → next is `00004`). If no migrations exist, start at `00001`
4. **List `db/queries/*.sql`** — inventory existing query files and their names
5. **Grep existing migrations for `CREATE TABLE` statements** — build awareness of existing tables, their columns, and foreign key targets. This prevents generating FKs to nonexistent tables
6. **Read all three data files:**
   - `skills/forge-migration/data/type-mappings.md`
   - `skills/forge-migration/data/patterns.md`
   - `skills/forge-migration/data/query-templates.md`

---

## Phase 1: Plan

Parse `$ARGUMENTS` to determine the operation:

### New Table (CREATE TABLE)
- Infer table name from description (pluralize if needed: "user" → "users")
- Map described columns to SQL types using `data/type-mappings.md` column name inference rules
- Auto-include these columns (do NOT ask, they are always present on new tables):
  - `id VARCHAR(26) PRIMARY KEY` (first column)
  - `created_at TIMESTAMPTZ NOT NULL DEFAULT now()` (second-to-last)
  - `updated_at TIMESTAMPTZ NOT NULL DEFAULT now()` (last)
- Auto-include the `updated_at` trigger function and trigger
- Detect patterns from description using `data/patterns.md`:
  - "soft delete" or "deletable" → add `deleted_at` column + partial index
  - "status" or "enum" → add enum type if appropriate
  - References to other tables → add FK constraints + indexes
  - "many-to-many" or "junction" → junction table pattern
- Propose standard CRUD queries using `data/query-templates.md`:
  - Always: `GetByID`, `List` (paginated), `Create`, `Update`, `Delete`
  - If soft delete: replace `Delete` with `SoftDelete`, add `ListActive`
  - If unique fields exist: add `GetBy<Field>` for each
  - If FK fields exist: add `ListBy<FK>` for each
  - If table has `name` or `title` or searchable text: add `Search`
  - If existing query file found: parse existing query names and propose only missing ones

### Alter Table (ALTER TABLE)
- Detect operation: add column, drop column, rename column, add constraint, add index
- Generate appropriate ALTER TABLE migration
- Propose any new queries that make sense for the added columns
- If query file exists, append only new queries

### Decision: target file paths
- Migration file: `db/migrations/NNNNN_<descriptive_name>.sql` (zero-padded to 5 digits)
- Query file: `db/queries/<table_name>.sql` (new or existing)

---

## Phase 2: Ask (NEVER skip this phase)

Present the full plan to the user using clear formatting:

### Migration SQL Preview
Show the complete Up and Down SQL that will be generated.

### Query List
Show a table of proposed queries:
| Query Name | Annotation | Description |
|---|---|---|
| `GetUserByID` | `:one` | Fetch single user by ID |
| ... | ... | ... |

### Target Files
- Migration: `db/migrations/NNNNN_<name>.sql`
- Queries: `db/queries/<table>.sql` (new file / append to existing)

### Ask for Confirmation
Let the user:
- Adjust columns (add, remove, change types)
- Add or remove queries
- Change naming
- Approve to proceed

**Do not generate any files until the user confirms.**

---

## Phase 3: Generate

### Migration File

Write `db/migrations/NNNNN_<name>.sql` with this structure:

```sql
-- +goose Up

<SQL statements>

-- +goose Down

<Reverse SQL statements>
```

Rules:
- Up section creates/alters; Down section reverses exactly
- For new tables: Down drops trigger first, then table, then any enum types
- For alter tables: Down reverses each alteration
- Include `updated_at` trigger for every new table (see `data/patterns.md` for the trigger pattern)
- Use `IF NOT EXISTS` / `IF EXISTS` where appropriate in Down migrations

### Query File

**New table** — Write `db/queries/<table>.sql` with all confirmed queries.

**Existing table** — Use the `Edit` tool to append new queries to the end of the existing `db/queries/<table>.sql` file. Never overwrite existing queries.

Query rules (from `data/query-templates.md`):
- Every query starts with `-- name: <QueryName> :<annotation>`
- `id` is always `$1` in INSERT (caller provides via `id.New()`)
- `id` is never in UPDATE SET clause
- `created_at` is never in UPDATE SET clause
- `updated_at` is explicitly set to `now()` in UPDATE
- List queries use `LIMIT $N OFFSET $M` for pagination
- Use `sqlc.arg()` for named parameters when clarity helps

---

## Phase 4: Run & Verify

### Step 1: Generate repository code
```bash
go tool sqlc generate -f db/sqlc.yml
```

If sqlc fails:
1. Read the error output carefully
2. Fix the SQL that caused the error (migration or query file)
3. Re-run `go tool sqlc generate -f db/sqlc.yml`
4. Repeat until clean

### Step 2: Verify compilation
```bash
go build -o /dev/null ./...
```

If build fails:
1. Read the error — usually a type mismatch or missing import
2. Fix the source
3. Re-run build
4. Repeat until clean

---

## Phase 5: Summary

After successful generation and verification, output:

### Files Created/Modified
List each file path with (created) or (modified) label.

### Generated Query Functions
Show a table of all query functions now available:
| Function | Signature | Description |
|---|---|---|
| `GetUserByID` | `(ctx, id string) (User, error)` | Fetch user by ID |
| ... | ... | ... |

### Reminder
> Migrations auto-run on app start via `db.Open()` — no manual step needed in dev.

### Next Steps
- Create a handler that uses `repository.New(pool)` to access the generated methods
- List the available repository methods the handler can call

---

## Important Rules

- **Forge IDs**: Always `VARCHAR(26)` for primary keys and foreign keys — never UUID, SERIAL, or IDENTITY
- **Timestamps**: Always `TIMESTAMPTZ` with `NOT NULL DEFAULT now()` for `created_at` and `updated_at`
- **Trigger**: Every new table gets an `updated_at` trigger
- **Naming**: Snake_case for SQL (table names, column names), PascalCase for query names
- **Semicolons**: Forge validation tags use semicolons as separators — but SQL uses standard syntax
- **No binary artifacts**: `go build -o /dev/null` — never leave binaries
- **Edit, don't overwrite**: When adding queries to existing files, use `Edit` to append
