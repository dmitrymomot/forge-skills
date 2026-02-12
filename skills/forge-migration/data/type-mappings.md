# PostgreSQL ↔ Go Type Mappings

Reference for the forge-migration skill. Used during Phase 1 to infer column types and during Phase 3 to predict generated Go types.

---

## PostgreSQL → Go Type Mapping

These reflect the standard sqlc overrides configured by `forge-init`:

| PostgreSQL Type | Nullable? | Go Type | Notes |
|---|---|---|---|
| `VARCHAR(n)` | NOT NULL | `string` | |
| `VARCHAR(n)` | NULL | `*string` | |
| `TEXT` | NOT NULL | `string` | |
| `TEXT` | NULL | `*string` | |
| `INTEGER` | NOT NULL | `int32` | |
| `INTEGER` | NULL | `*int32` | |
| `BIGINT` | NOT NULL | `int64` | |
| `BIGINT` | NULL | `*int64` | |
| `SMALLINT` | NOT NULL | `int16` | |
| `BOOLEAN` | NOT NULL | `bool` | |
| `BOOLEAN` | NULL | `*bool` | |
| `NUMERIC(p,s)` | NOT NULL | `string` | sqlc maps numeric to string by default |
| `NUMERIC(p,s)` | NULL | `*string` | |
| `TIMESTAMPTZ` | NOT NULL | `time.Time` | |
| `TIMESTAMPTZ` | NULL | `*time.Time` | Used for soft delete `deleted_at` |
| `TIMESTAMP` | NOT NULL | `time.Time` | Prefer TIMESTAMPTZ |
| `TIMESTAMP` | NULL | `*time.Time` | |
| `JSONB` | NOT NULL | `json.RawMessage` | |
| `JSONB` | NULL | `json.RawMessage` | sqlc maps both to RawMessage |
| `UUID` | NOT NULL | `string` | Avoid — use VARCHAR(26) for Forge IDs |
| `BYTEA` | NOT NULL | `[]byte` | |
| Custom ENUM | NOT NULL | `string` | sqlc generates string type for enums |

---

## Column Name Inference Rules

When the user describes columns by name only (no explicit type), infer the SQL type:

### Identity & References
| Column Name Pattern | SQL Type | Constraints |
|---|---|---|
| `id` | `VARCHAR(26)` | `PRIMARY KEY` |
| `*_id` (e.g., `user_id`, `org_id`) | `VARCHAR(26)` | `NOT NULL` — FK candidate |

### Text Fields
| Column Name Pattern | SQL Type | Constraints |
|---|---|---|
| `email` | `VARCHAR(255)` | `NOT NULL UNIQUE` |
| `name`, `first_name`, `last_name`, `title` | `VARCHAR(255)` | `NOT NULL` |
| `display_name`, `username` | `VARCHAR(255)` | `NOT NULL UNIQUE` |
| `description`, `content`, `body`, `bio` | `TEXT` | `NOT NULL DEFAULT ''` |
| `slug` | `VARCHAR(255)` | `NOT NULL UNIQUE` |
| `*_url` (e.g., `avatar_url`, `website_url`) | `TEXT` | `NOT NULL DEFAULT ''` |
| `phone`, `phone_number` | `VARCHAR(50)` | `NOT NULL DEFAULT ''` |
| `password_hash`, `*_hash` | `TEXT` | `NOT NULL` |
| `token`, `*_token` | `TEXT` | `NOT NULL` |

### Status & Flags
| Column Name Pattern | SQL Type | Constraints |
|---|---|---|
| `status` | `VARCHAR(50)` | `NOT NULL DEFAULT 'active'` |
| `role` | `VARCHAR(50)` | `NOT NULL DEFAULT 'member'` |
| `type`, `kind`, `category` | `VARCHAR(50)` | `NOT NULL` |
| `is_*`, `has_*` (e.g., `is_active`, `has_verified`) | `BOOLEAN` | `NOT NULL DEFAULT false` |

### Numeric Fields
| Column Name Pattern | SQL Type | Constraints |
|---|---|---|
| `price`, `amount`, `total`, `balance`, `cost` | `NUMERIC(10,2)` | `NOT NULL DEFAULT 0` |
| `count`, `quantity`, `position`, `order` | `INTEGER` | `NOT NULL DEFAULT 0` |
| `sort_order`, `priority`, `rank` | `INTEGER` | `NOT NULL DEFAULT 0` |

### JSON Fields
| Column Name Pattern | SQL Type | Constraints |
|---|---|---|
| `metadata`, `settings`, `config`, `preferences` | `JSONB` | `NOT NULL DEFAULT '{}'::jsonb` |
| `tags`, `labels` | `JSONB` | `NOT NULL DEFAULT '[]'::jsonb` |
| `data`, `payload`, `attrs`, `attributes` | `JSONB` | `NOT NULL DEFAULT '{}'::jsonb` |

### Timestamps
| Column Name Pattern | SQL Type | Constraints |
|---|---|---|
| `created_at` | `TIMESTAMPTZ` | `NOT NULL DEFAULT now()` — auto-included |
| `updated_at` | `TIMESTAMPTZ` | `NOT NULL DEFAULT now()` — auto-included |
| `deleted_at` | `TIMESTAMPTZ` | `NULL` — soft delete pattern |
| `*_at` (e.g., `verified_at`, `published_at`, `expires_at`) | `TIMESTAMPTZ` | `NULL` |

---

## Forge ID Rules

Forge uses ULIDs stored as `VARCHAR(26)`. Never use these alternatives:

| Avoid | Use Instead | Reason |
|---|---|---|
| `UUID` | `VARCHAR(26)` | Forge IDs are ULIDs via `pkg/id` |
| `SERIAL` / `BIGSERIAL` | `VARCHAR(26)` | No auto-increment — caller generates IDs |
| `IDENTITY` | `VARCHAR(26)` | Same as SERIAL — not used in Forge |
| `uuid_generate_v4()` | Caller uses `id.New()` | ID generation is application-side |

### ID in INSERT Queries
The `id` is always the first parameter (`$1`) in INSERT statements. The caller generates it:
```go
id := id.New() // from github.com/dmitrymomot/forge/pkg/id
err := queries.CreateUser(ctx, repository.CreateUserParams{
    ID:    id,
    Name:  "Alice",
    Email: "alice@example.com",
})
```
