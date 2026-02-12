# Database: Migrations, Queries & Type Mappings

Loaded when the feature involves database changes (tables, columns, queries).

---

## Migration Patterns

### Standard Table Skeleton

Every new table starts with this structure. The `updated_at` trigger is mandatory.

#### Up
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

#### Down
```sql
DROP TRIGGER IF EXISTS set_table_name_updated_at ON table_name;
DROP TABLE IF EXISTS table_name;
-- Only drop the function if no other tables use it:
-- DROP FUNCTION IF EXISTS set_updated_at();
```

The `set_updated_at()` function is shared across tables. Only drop it in Down if this is the very first table.

---

### Soft Delete

Additional column:
```sql
deleted_at TIMESTAMPTZ NULL
```

Partial index:
```sql
CREATE INDEX idx_table_name_active ON table_name (id) WHERE deleted_at IS NULL;
```

Down:
```sql
DROP INDEX IF EXISTS idx_table_name_active;
ALTER TABLE table_name DROP COLUMN IF EXISTS deleted_at;
```

Impact: Replace `DELETE` with `UPDATE SET deleted_at = now()`. All SELECTs add `WHERE deleted_at IS NULL`.

---

### Enum Type

```sql
CREATE TYPE enum_type_name AS ENUM ('value1', 'value2', 'value3');
```

Usage: `status enum_type_name NOT NULL DEFAULT 'value1'`

Down: Drop column/table using the enum before `DROP TYPE IF EXISTS enum_type_name;`

**Prefer VARCHAR with DEFAULT** for most cases — enums require migrations to add values.

---

### Foreign Key

```sql
parent_id VARCHAR(26) NOT NULL REFERENCES parent_table(id) ON DELETE CASCADE,
```

Always add an index:
```sql
CREATE INDEX idx_table_name_parent_id ON table_name (parent_id);
```

| ON DELETE | When to Use |
|---|---|
| `CASCADE` | Child has no meaning without parent |
| `SET NULL` | Child can exist independently (nullable FK) |
| `RESTRICT` | Prevent deletion if children exist |

---

### Junction Table (Many-to-Many)

```sql
CREATE TABLE left_right (
    left_id VARCHAR(26) NOT NULL REFERENCES left_table(id) ON DELETE CASCADE,
    right_id VARCHAR(26) NOT NULL REFERENCES right_table(id) ON DELETE CASCADE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    PRIMARY KEY (left_id, right_id)
);
CREATE INDEX idx_left_right_right_id ON left_right (right_id);
```

No `id` column, no `updated_at`. Add extra columns if the relationship has attributes.

---

### Other Patterns

**Unique Constraint:**
```sql
ALTER TABLE t ADD CONSTRAINT uq_t_col UNIQUE (col);
```

**GIN Index for JSONB:**
```sql
CREATE INDEX idx_t_metadata ON t USING GIN (metadata);
```

**CHECK Constraint:**
```sql
ALTER TABLE t ADD CONSTRAINT chk_t_amount CHECK (amount >= 0);
ALTER TABLE t ADD CONSTRAINT chk_t_status CHECK (status IN ('active', 'inactive'));
```

---

### ALTER TABLE Patterns

```sql
-- Add Column
ALTER TABLE t ADD COLUMN col TYPE CONSTRAINTS;
-- Drop Column
ALTER TABLE t DROP COLUMN IF EXISTS col;
-- Rename Column
ALTER TABLE t RENAME COLUMN old TO new;
-- Change Type
ALTER TABLE t ALTER COLUMN col TYPE NEW_TYPE USING col::NEW_TYPE;
-- Add/Drop NOT NULL
ALTER TABLE t ALTER COLUMN col SET NOT NULL;
ALTER TABLE t ALTER COLUMN col DROP NOT NULL;
-- Add/Drop Default
ALTER TABLE t ALTER COLUMN col SET DEFAULT 'value';
ALTER TABLE t ALTER COLUMN col DROP DEFAULT;
-- Add/Drop Index
CREATE INDEX idx_name ON t (col);
DROP INDEX IF EXISTS idx_name;
```

Each ALTER needs both Up and Down in the migration file.

---

## sqlc Query Templates

### Annotation Reference

| Annotation | Returns | Use When |
|---|---|---|
| `:one` | Single struct | SELECT returning one row |
| `:many` | Slice of structs | SELECT returning multiple rows |
| `:exec` | `error` only | INSERT/UPDATE/DELETE, no result needed |
| `:execresult` | `sql.Result, error` | Need `RowsAffected()` |
| `:execrows` | `int64, error` | Only need affected count |

### Standard CRUD

```sql
-- name: Get<Entity>ByID :one
SELECT * FROM table_name WHERE id = $1;

-- name: List<Entity>s :many
SELECT * FROM table_name ORDER BY created_at DESC LIMIT $1 OFFSET $2;

-- name: Create<Entity> :exec
INSERT INTO table_name (id, col1, col2, created_at, updated_at)
VALUES ($1, $2, $3, now(), now());

-- name: Update<Entity> :exec
UPDATE table_name SET col1 = $2, col2 = $3, updated_at = now() WHERE id = $1;

-- name: Delete<Entity> :exec
DELETE FROM table_name WHERE id = $1;
```

**Rules:**
- `id` is always `$1` in INSERT — caller generates via `id.New()`
- `created_at`/`updated_at` use `now()` — never parameters
- `id` and `created_at` are never in UPDATE SET
- `updated_at` is explicitly `now()` in UPDATE

### Supplemental Queries

```sql
-- GetBy unique field
-- name: Get<Entity>By<Field> :one
SELECT * FROM table_name WHERE field = $1;

-- ListBy foreign key
-- name: List<Entity>sBy<Parent> :many
SELECT * FROM table_name WHERE parent_id = $1 ORDER BY created_at DESC LIMIT $2 OFFSET $3;

-- Count
-- name: Count<Entity>s :one
SELECT count(*) FROM table_name;

-- Exists
-- name: <Entity>Exists :one
SELECT EXISTS(SELECT 1 FROM table_name WHERE id = $1);
```

### Soft Delete Queries

```sql
-- name: SoftDelete<Entity> :exec
UPDATE table_name SET deleted_at = now(), updated_at = now() WHERE id = $1;

-- name: Restore<Entity> :exec
UPDATE table_name SET deleted_at = NULL, updated_at = now() WHERE id = $1;

-- name: ListActive<Entity>s :many
SELECT * FROM table_name WHERE deleted_at IS NULL ORDER BY created_at DESC LIMIT $1 OFFSET $2;

-- name: Get<Entity>ByID :one (soft-delete aware)
SELECT * FROM table_name WHERE id = $1 AND deleted_at IS NULL;
```

### Search Queries

```sql
-- name: Search<Entity>s :many
SELECT * FROM table_name
WHERE name ILIKE '%' || $1 || '%'
ORDER BY created_at DESC LIMIT $2 OFFSET $3;
```

### Junction Table Queries

```sql
-- name: Add<Left><Right> :exec
INSERT INTO left_right (left_id, right_id) VALUES ($1, $2) ON CONFLICT DO NOTHING;

-- name: Remove<Left><Right> :exec
DELETE FROM left_right WHERE left_id = $1 AND right_id = $2;

-- name: List<Right>sBy<Left> :many
SELECT r.* FROM right_table r
JOIN left_right lr ON lr.right_id = r.id
WHERE lr.left_id = $1 ORDER BY r.created_at DESC LIMIT $2 OFFSET $3;
```

### Query Naming Conventions

| Pattern | Format | Example |
|---|---|---|
| Get single | `Get<Entity>By<Field>` | `GetUserByEmail` |
| List multiple | `List<Entity>s` | `ListUsers` |
| List filtered | `List<Entity>sBy<Filter>` | `ListProjectsByOwner` |
| Create | `Create<Entity>` | `CreateUser` |
| Update | `Update<Entity>` | `UpdateUser` |
| Delete | `Delete<Entity>` | `DeleteUser` |
| Soft delete | `SoftDelete<Entity>` | `SoftDeleteUser` |
| Count | `Count<Entity>s` | `CountUsers` |
| Exists | `<Entity>Exists` | `UserExists` |
| Search | `Search<Entity>s` | `SearchUsers` |
| Junction | `Add/Remove/List<B>sBy<A>` | `AddUserRole` |

Entity: singular PascalCase (`User`). Table: plural snake_case (`users`).

---

## PostgreSQL to Go Type Mappings

| PostgreSQL Type | Nullable? | Go Type |
|---|---|---|
| `VARCHAR(n)` | NOT NULL | `string` |
| `VARCHAR(n)` | NULL | `*string` |
| `TEXT` | NOT NULL | `string` |
| `TEXT` | NULL | `*string` |
| `INTEGER` | NOT NULL | `int32` |
| `INTEGER` | NULL | `*int32` |
| `BIGINT` | NOT NULL | `int64` |
| `BIGINT` | NULL | `*int64` |
| `SMALLINT` | NOT NULL | `int16` |
| `BOOLEAN` | NOT NULL | `bool` |
| `BOOLEAN` | NULL | `*bool` |
| `NUMERIC(p,s)` | NOT NULL | `string` |
| `NUMERIC(p,s)` | NULL | `*string` |
| `TIMESTAMPTZ` | NOT NULL | `time.Time` |
| `TIMESTAMPTZ` | NULL | `*time.Time` |
| `JSONB` | NOT NULL/NULL | `json.RawMessage` |
| `BYTEA` | NOT NULL | `[]byte` |
| Custom ENUM | NOT NULL | `string` |

---

## Column Name Inference Rules

When user describes columns by name only, infer SQL types:

### Identity & References
| Pattern | SQL Type | Constraints |
|---|---|---|
| `id` | `VARCHAR(26)` | `PRIMARY KEY` |
| `*_id` | `VARCHAR(26)` | `NOT NULL` — FK candidate |

### Text Fields
| Pattern | SQL Type | Constraints |
|---|---|---|
| `email` | `VARCHAR(255)` | `NOT NULL UNIQUE` |
| `name`, `first_name`, `last_name`, `title` | `VARCHAR(255)` | `NOT NULL` |
| `display_name`, `username` | `VARCHAR(255)` | `NOT NULL UNIQUE` |
| `description`, `content`, `body`, `bio` | `TEXT` | `NOT NULL DEFAULT ''` |
| `slug` | `VARCHAR(255)` | `NOT NULL UNIQUE` |
| `*_url` | `TEXT` | `NOT NULL DEFAULT ''` |
| `phone`, `phone_number` | `VARCHAR(50)` | `NOT NULL DEFAULT ''` |
| `password_hash`, `*_hash` | `TEXT` | `NOT NULL` |
| `token`, `*_token` | `TEXT` | `NOT NULL` |

### Status & Flags
| Pattern | SQL Type | Constraints |
|---|---|---|
| `status` | `VARCHAR(50)` | `NOT NULL DEFAULT 'active'` |
| `role` | `VARCHAR(50)` | `NOT NULL DEFAULT 'member'` |
| `type`, `kind`, `category` | `VARCHAR(50)` | `NOT NULL` |
| `is_*`, `has_*` | `BOOLEAN` | `NOT NULL DEFAULT false` |

### Numeric Fields
| Pattern | SQL Type | Constraints |
|---|---|---|
| `price`, `amount`, `total`, `balance`, `cost` | `NUMERIC(10,2)` | `NOT NULL DEFAULT 0` |
| `count`, `quantity`, `position`, `order` | `INTEGER` | `NOT NULL DEFAULT 0` |
| `sort_order`, `priority`, `rank` | `INTEGER` | `NOT NULL DEFAULT 0` |

### JSON Fields
| Pattern | SQL Type | Constraints |
|---|---|---|
| `metadata`, `settings`, `config`, `preferences` | `JSONB` | `NOT NULL DEFAULT '{}'::jsonb` |
| `tags`, `labels` | `JSONB` | `NOT NULL DEFAULT '[]'::jsonb` |
| `data`, `payload`, `attrs`, `attributes` | `JSONB` | `NOT NULL DEFAULT '{}'::jsonb` |

### Timestamps
| Pattern | SQL Type | Constraints |
|---|---|---|
| `created_at` | `TIMESTAMPTZ` | `NOT NULL DEFAULT now()` — auto-included |
| `updated_at` | `TIMESTAMPTZ` | `NOT NULL DEFAULT now()` — auto-included |
| `deleted_at` | `TIMESTAMPTZ` | `NULL` — soft delete |
| `*_at` | `TIMESTAMPTZ` | `NULL` |

---

## Repository Usage

After `go tool sqlc generate`, use the generated repository:

```go
import "myapp/internal/repository"

// In handler constructor
type PostHandler struct {
    repo *repository.Queries
}

func NewPostHandler(pool *pgxpool.Pool) *PostHandler {
    return &PostHandler{repo: repository.New(pool)}
}

// In handler methods
func (h *PostHandler) create(c forge.Context) error {
    post, err := h.repo.CreatePost(c, repository.CreatePostParams{
        ID:    id.New(),
        Title: req.Title,
        Body:  req.Body,
    })
    if err != nil {
        return c.Error(err)
    }
    return c.JSON(http.StatusCreated, post)
}
```

Generate command: `go tool sqlc generate -f db/sqlc.yml`
