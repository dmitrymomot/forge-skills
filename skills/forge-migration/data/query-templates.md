# sqlc Query Templates

Query templates for the forge-migration skill. Used during Phase 1 to propose queries and Phase 3 to generate them.

---

## sqlc Annotation Reference

| Annotation | Returns | Use When |
|---|---|---|
| `:one` | Single struct | SELECT that returns exactly one row (or error) |
| `:many` | Slice of structs | SELECT that returns multiple rows |
| `:exec` | `error` only | INSERT, UPDATE, DELETE where you don't need result info |
| `:execresult` | `sql.Result, error` | When you need `RowsAffected()` or `LastInsertId()` |
| `:execrows` | `int64, error` | When you only need the count of affected rows |

---

## Standard CRUD Queries

### GetByID

```sql
-- name: Get<Entity>ByID :one
SELECT * FROM table_name
WHERE id = $1;
```

### List (Paginated)

```sql
-- name: List<Entity>s :many
SELECT * FROM table_name
ORDER BY created_at DESC
LIMIT $1 OFFSET $2;
```

### Create

```sql
-- name: Create<Entity> :exec
INSERT INTO table_name (
    id,
    column1,
    column2,
    created_at,
    updated_at
) VALUES (
    $1, $2, $3, now(), now()
);
```

**Rules:**
- `id` is always `$1` — caller generates it via `id.New()`
- `created_at` and `updated_at` use `now()` — never passed as parameters
- List all columns explicitly — never use `DEFAULT` for `id`

### Update

```sql
-- name: Update<Entity> :exec
UPDATE table_name
SET
    column1 = $2,
    column2 = $3,
    updated_at = now()
WHERE id = $1;
```

**Rules:**
- `id` is in WHERE, never in SET
- `created_at` is never in SET
- `updated_at` is explicitly set to `now()`
- `$1` is always the `id` (WHERE clause)

### Delete (Hard)

```sql
-- name: Delete<Entity> :exec
DELETE FROM table_name
WHERE id = $1;
```

---

## Supplemental Queries

### GetBy<UniqueField>

For each column with a UNIQUE constraint:

```sql
-- name: Get<Entity>By<Field> :one
SELECT * FROM table_name
WHERE field_name = $1;
```

Examples:
```sql
-- name: GetUserByEmail :one
SELECT * FROM users WHERE email = $1;

-- name: GetUserByUsername :one
SELECT * FROM users WHERE username = $1;
```

### ListBy<ForeignKey>

For each foreign key column:

```sql
-- name: List<Entity>sBy<Parent> :many
SELECT * FROM table_name
WHERE parent_id = $1
ORDER BY created_at DESC
LIMIT $2 OFFSET $3;
```

### Count

```sql
-- name: Count<Entity>s :one
SELECT count(*) FROM table_name;
```

### CountBy<ForeignKey>

```sql
-- name: Count<Entity>sBy<Parent> :one
SELECT count(*) FROM table_name
WHERE parent_id = $1;
```

### Exists

```sql
-- name: <Entity>Exists :one
SELECT EXISTS(SELECT 1 FROM table_name WHERE id = $1);
```

---

## Soft Delete Queries

Use these instead of hard delete when the table has a `deleted_at` column.

### SoftDelete

```sql
-- name: SoftDelete<Entity> :exec
UPDATE table_name
SET deleted_at = now(), updated_at = now()
WHERE id = $1;
```

### Restore (optional)

```sql
-- name: Restore<Entity> :exec
UPDATE table_name
SET deleted_at = NULL, updated_at = now()
WHERE id = $1;
```

### ListActive (excludes soft-deleted)

```sql
-- name: ListActive<Entity>s :many
SELECT * FROM table_name
WHERE deleted_at IS NULL
ORDER BY created_at DESC
LIMIT $1 OFFSET $2;
```

### GetByID (soft-delete aware)

When soft delete is enabled, the standard GetByID should exclude deleted records:

```sql
-- name: Get<Entity>ByID :one
SELECT * FROM table_name
WHERE id = $1 AND deleted_at IS NULL;
```

---

## Search Queries

For tables with searchable text fields (`name`, `title`, `description`):

### Simple ILIKE Search

```sql
-- name: Search<Entity>s :many
SELECT * FROM table_name
WHERE name ILIKE '%' || $1 || '%'
ORDER BY created_at DESC
LIMIT $2 OFFSET $3;
```

### Multi-field Search

```sql
-- name: Search<Entity>s :many
SELECT * FROM table_name
WHERE
    name ILIKE '%' || $1 || '%'
    OR description ILIKE '%' || $1 || '%'
ORDER BY created_at DESC
LIMIT $2 OFFSET $3;
```

---

## Junction Table Queries

For many-to-many relationships:

### Add Relation

```sql
-- name: Add<Left><Right> :exec
INSERT INTO left_right (left_id, right_id)
VALUES ($1, $2)
ON CONFLICT DO NOTHING;
```

### Remove Relation

```sql
-- name: Remove<Left><Right> :exec
DELETE FROM left_right
WHERE left_id = $1 AND right_id = $2;
```

### List by Left

```sql
-- name: List<Right>sBy<Left> :many
SELECT r.* FROM right_table r
JOIN left_right lr ON lr.right_id = r.id
WHERE lr.left_id = $1
ORDER BY r.created_at DESC
LIMIT $2 OFFSET $3;
```

### List by Right

```sql
-- name: List<Left>sBy<Right> :many
SELECT l.* FROM left_table l
JOIN left_right lr ON lr.left_id = l.id
WHERE lr.right_id = $1
ORDER BY l.created_at DESC
LIMIT $2 OFFSET $3;
```

---

## Query Naming Conventions

| Pattern | Format | Example |
|---|---|---|
| Get single | `Get<Entity>By<Field>` | `GetUserByEmail` |
| List multiple | `List<Entity>s` | `ListUsers` |
| List filtered | `List<Entity>sBy<Filter>` | `ListProjectsByOwner` |
| List active | `ListActive<Entity>s` | `ListActiveUsers` |
| Create | `Create<Entity>` | `CreateUser` |
| Update | `Update<Entity>` | `UpdateUser` |
| Delete | `Delete<Entity>` | `DeleteUser` |
| Soft delete | `SoftDelete<Entity>` | `SoftDeleteUser` |
| Restore | `Restore<Entity>` | `RestoreUser` |
| Count | `Count<Entity>s` | `CountUsers` |
| Count filtered | `Count<Entity>sBy<Filter>` | `CountProjectsByOwner` |
| Exists | `<Entity>Exists` | `UserExists` |
| Search | `Search<Entity>s` | `SearchUsers` |
| Junction add | `Add<Left><Right>` | `AddUserRole` |
| Junction remove | `Remove<Left><Right>` | `RemoveUserRole` |
| Junction list | `List<B>sBy<A>` | `ListRolesByUser` |

### Entity Name Rules
- Entity names are **singular PascalCase**: `User`, `Project`, `TeamMember`
- Table names are **plural snake_case**: `users`, `projects`, `team_members`
- Convert table → entity: `team_members` → `TeamMember`
