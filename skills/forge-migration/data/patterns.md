# Migration Patterns

Copy-pasteable migration blocks for the forge-migration skill. Each pattern includes both Up and Down SQL.

---

## 1. Standard Table Skeleton

Every new table starts with this structure. The `updated_at` trigger is mandatory.

### Up
```sql
-- Create the trigger function if it doesn't exist yet
CREATE OR REPLACE FUNCTION set_updated_at()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = now();
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TABLE table_name (
    id VARCHAR(26) PRIMARY KEY,
    -- user columns here --
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TRIGGER set_table_name_updated_at
    BEFORE UPDATE ON table_name
    FOR EACH ROW
    EXECUTE FUNCTION set_updated_at();
```

### Down
```sql
DROP TRIGGER IF EXISTS set_table_name_updated_at ON table_name;
DROP TABLE IF EXISTS table_name;
-- Only drop the function if no other tables use it:
-- DROP FUNCTION IF EXISTS set_updated_at();
```

**Note:** The `set_updated_at()` function is shared across tables. Only drop it in the Down migration if this is the very first table being created (i.e., no other tables depend on it). Otherwise, leave it.

---

## 2. Soft Delete

Add to any table that should support soft deletion instead of hard delete.

### Additional Column
```sql
deleted_at TIMESTAMPTZ NULL
```

### Partial Index (Up)
```sql
CREATE INDEX idx_table_name_active ON table_name (id)
    WHERE deleted_at IS NULL;
```

### Down
```sql
DROP INDEX IF EXISTS idx_table_name_active;
ALTER TABLE table_name DROP COLUMN IF EXISTS deleted_at;
```

### Query Impact
- Replace `DELETE` with `UPDATE ... SET deleted_at = now()`
- All SELECT queries should include `WHERE deleted_at IS NULL` (or provide both filtered and unfiltered variants)

---

## 3. Enum Type

Use PostgreSQL enums for columns with a fixed, small set of values.

### Up
```sql
CREATE TYPE enum_type_name AS ENUM ('value1', 'value2', 'value3');
```

### Usage in Table
```sql
status enum_type_name NOT NULL DEFAULT 'value1'
```

### Down
```sql
-- Must drop column/table using the enum before dropping the type
DROP TYPE IF EXISTS enum_type_name;
```

**When to use enums vs VARCHAR:**
- Enum: Fixed set of values that rarely changes (e.g., `order_status`: pending, processing, shipped, delivered)
- VARCHAR with CHECK: Values that may change more frequently (e.g., user roles that admins can configure)
- VARCHAR with DEFAULT: Simple status fields (e.g., `status VARCHAR(50) NOT NULL DEFAULT 'active'`)

Prefer VARCHAR with DEFAULT for most cases — enums require migrations to add new values.

---

## 4. Foreign Key

### Column + Constraint + Index (Up)
```sql
parent_id VARCHAR(26) NOT NULL REFERENCES parent_table(id) ON DELETE CASCADE,
```

### Index (always add for FK columns)
```sql
CREATE INDEX idx_table_name_parent_id ON table_name (parent_id);
```

### Down
```sql
DROP INDEX IF EXISTS idx_table_name_parent_id;
-- FK constraint is dropped automatically with the column or table
```

### ON DELETE Options
| Option | When to Use |
|---|---|
| `CASCADE` | Child has no meaning without parent (e.g., comments on a post) |
| `SET NULL` | Child can exist independently (requires nullable FK) |
| `RESTRICT` | Prevent deletion if children exist (e.g., user with active subscriptions) |

---

## 5. Junction Table (Many-to-Many)

### Up
```sql
CREATE TABLE left_right (
    left_id VARCHAR(26) NOT NULL REFERENCES left_table(id) ON DELETE CASCADE,
    right_id VARCHAR(26) NOT NULL REFERENCES right_table(id) ON DELETE CASCADE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    PRIMARY KEY (left_id, right_id)
);

CREATE INDEX idx_left_right_right_id ON left_right (right_id);
```

### Down
```sql
DROP TABLE IF EXISTS left_right;
```

**Notes:**
- Junction tables use a composite primary key — no `id` column, no `updated_at`
- The reverse index on `right_id` enables efficient lookups in both directions
- Add extra columns (e.g., `role`, `sort_order`) if the relationship has attributes

---

## 6. Unique Constraint

### Up
```sql
ALTER TABLE table_name ADD CONSTRAINT uq_table_name_column UNIQUE (column_name);
```

### Composite Unique
```sql
ALTER TABLE table_name ADD CONSTRAINT uq_table_name_col1_col2 UNIQUE (col1, col2);
```

### Down
```sql
ALTER TABLE table_name DROP CONSTRAINT IF EXISTS uq_table_name_column;
```

---

## 7. GIN Index for JSONB

Use for JSONB columns that will be queried with `@>`, `?`, or `?|` operators.

### Up
```sql
CREATE INDEX idx_table_name_metadata ON table_name USING GIN (metadata);
```

### Down
```sql
DROP INDEX IF EXISTS idx_table_name_metadata;
```

---

## 8. CHECK Constraint

### Up
```sql
ALTER TABLE table_name ADD CONSTRAINT chk_table_name_amount CHECK (amount >= 0);
```

### String Enum via CHECK
```sql
ALTER TABLE table_name ADD CONSTRAINT chk_table_name_status
    CHECK (status IN ('active', 'inactive', 'suspended'));
```

### Down
```sql
ALTER TABLE table_name DROP CONSTRAINT IF EXISTS chk_table_name_amount;
```

---

## 9. ALTER TABLE Patterns

### Add Column
```sql
-- +goose Up
ALTER TABLE table_name ADD COLUMN column_name TYPE CONSTRAINTS;

-- +goose Down
ALTER TABLE table_name DROP COLUMN IF EXISTS column_name;
```

### Drop Column
```sql
-- +goose Up
ALTER TABLE table_name DROP COLUMN IF EXISTS column_name;

-- +goose Down
ALTER TABLE table_name ADD COLUMN column_name TYPE CONSTRAINTS;
```

### Rename Column
```sql
-- +goose Up
ALTER TABLE table_name RENAME COLUMN old_name TO new_name;

-- +goose Down
ALTER TABLE table_name RENAME COLUMN new_name TO old_name;
```

### Change Column Type
```sql
-- +goose Up
ALTER TABLE table_name ALTER COLUMN column_name TYPE NEW_TYPE USING column_name::NEW_TYPE;

-- +goose Down
ALTER TABLE table_name ALTER COLUMN column_name TYPE OLD_TYPE USING column_name::OLD_TYPE;
```

### Add/Drop NOT NULL
```sql
-- +goose Up
ALTER TABLE table_name ALTER COLUMN column_name SET NOT NULL;

-- +goose Down
ALTER TABLE table_name ALTER COLUMN column_name DROP NOT NULL;
```

### Add/Drop Default
```sql
-- +goose Up
ALTER TABLE table_name ALTER COLUMN column_name SET DEFAULT 'value';

-- +goose Down
ALTER TABLE table_name ALTER COLUMN column_name DROP DEFAULT;
```

### Add/Drop Constraint
```sql
-- +goose Up
ALTER TABLE table_name ADD CONSTRAINT constraint_name CONSTRAINT_DEFINITION;

-- +goose Down
ALTER TABLE table_name DROP CONSTRAINT IF EXISTS constraint_name;
```

### Add/Drop Index
```sql
-- +goose Up
CREATE INDEX idx_name ON table_name (column_name);

-- +goose Down
DROP INDEX IF EXISTS idx_name;
```

### Rename Table
```sql
-- +goose Up
ALTER TABLE old_name RENAME TO new_name;

-- +goose Down
ALTER TABLE new_name RENAME TO old_name;
```
