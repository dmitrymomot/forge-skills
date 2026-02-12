# Sessions Init — Generated Files

Generate these files when `sessions` is selected. Since `sessions` requires `db`, the `db/migrations/` and `db/queries/` directories already exist.

## A. Migration: `db/migrations/00002_users_and_sessions.sql`

```sql
-- +goose Up
-- +goose StatementBegin

CREATE TABLE users (
    id VARCHAR(26) PRIMARY KEY,
    email VARCHAR(255) NOT NULL UNIQUE,
    display_name TEXT NOT NULL DEFAULT '',
    language VARCHAR(10) NOT NULL DEFAULT 'en',
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    deleted_at TIMESTAMPTZ
);

CREATE TRIGGER set_users_updated_at
    BEFORE UPDATE ON users
    FOR EACH ROW
    EXECUTE FUNCTION update_updated_at_column();

CREATE TABLE sessions (
    id           TEXT PRIMARY KEY,
    token_hash   TEXT NOT NULL,
    user_id      TEXT,
    data         JSONB NOT NULL DEFAULT '{}',
    ip           TEXT NOT NULL DEFAULT '',
    user_agent   TEXT NOT NULL DEFAULT '',
    device       TEXT NOT NULL DEFAULT '',
    fingerprint  TEXT NOT NULL DEFAULT '',
    created_at   TIMESTAMPTZ NOT NULL DEFAULT now(),
    last_active_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    expires_at   TIMESTAMPTZ NOT NULL
);

CREATE INDEX idx_sessions_token_hash ON sessions(token_hash);
CREATE INDEX idx_sessions_user_id ON sessions(user_id) WHERE user_id IS NOT NULL;
CREATE INDEX idx_sessions_expires_at ON sessions(expires_at);

-- +goose StatementEnd

-- +goose Down
-- +goose StatementBegin
DROP TABLE IF EXISTS sessions;
DROP TABLE IF EXISTS users;
-- +goose StatementEnd
```

### Schema Notes

- `users.id` is `VARCHAR(26)` for ULID-based IDs from `pkg/id`.
- `sessions` columns match `forge.Session` struct fields exactly (verified against `internal/session.go`).
- `token_hash` index is **CRITICAL** — queried on every request for session lookup.
- `user_id` partial index excludes NULL (anonymous sessions) for efficiency.
- `update_updated_at_column()` trigger function comes from `00001_init.sql` (created by db subsystem).

---

## B. Queries: `db/queries/sessions.sql`

```sql
-- name: CreateSession :exec
INSERT INTO sessions (id, token_hash, user_id, data, ip, user_agent, device, fingerprint, created_at, last_active_at, expires_at)
VALUES ($1, $2, $3, $4, $5, $6, $7, $8, $9, $10, $11);

-- name: GetSessionByTokenHash :one
SELECT id, token_hash, user_id, data, ip, user_agent, device, fingerprint, created_at, last_active_at, expires_at
FROM sessions
WHERE token_hash = $1;

-- name: UpdateSession :exec
UPDATE sessions
SET user_id = $1, data = $2, ip = $3, user_agent = $4, device = $5, fingerprint = $6, last_active_at = $7, expires_at = $8
WHERE id = $9;

-- name: DeleteSession :exec
DELETE FROM sessions WHERE id = $1;

-- name: ListSessionsByUserID :many
SELECT id, token_hash, user_id, data, ip, user_agent, device, fingerprint, created_at, last_active_at, expires_at
FROM sessions
WHERE user_id = $1
ORDER BY last_active_at DESC;

-- name: CountSessionsByUserID :one
SELECT count(*) FROM sessions WHERE user_id = $1;

-- name: DeleteSessionsByUserID :exec
DELETE FROM sessions WHERE user_id = $1;

-- name: DeleteSessionsByUserIDExcept :exec
DELETE FROM sessions WHERE user_id = $1 AND id != $2;

-- name: DeleteOldestSessionByUserID :exec
DELETE FROM sessions
WHERE id = (
    SELECT id FROM sessions
    WHERE user_id = $1
    ORDER BY last_active_at ASC
    LIMIT 1
);

-- name: TouchSession :exec
UPDATE sessions SET last_active_at = $1 WHERE id = $2;
```

### Query Notes

- Each query maps to one method on the `forge.SessionStore` interface.
- `GetSessionByTokenHash` uses the `idx_sessions_token_hash` index — called on every request.
- `DeleteOldestSessionByUserID` uses a subquery to find and delete in one statement.
- `TouchSession` is a lightweight update for activity tracking (no full session reload).

---

## C. Adapter: `internal/sessionstore/store.go`

Replace `{{MODULE_PATH}}` with the actual Go module path.

```go
package sessionstore

import (
	"context"
	"encoding/json"
	"time"

	"github.com/dmitrymomot/forge"
	"github.com/jackc/pgx/v5"
	"{{MODULE_PATH}}/internal/repository"
)

// Compile-time check: Store implements forge.SessionStore.
var _ forge.SessionStore = (*Store)(nil)

// Store is a PostgreSQL-backed session store using sqlc-generated queries.
type Store struct {
	q *repository.Queries
}

// New creates a new session store backed by the given repository.
func New(q *repository.Queries) *Store {
	return &Store{q: q}
}

func (s *Store) Create(ctx context.Context, sess *forge.Session) error {
	data, err := json.Marshal(sess.Data)
	if err != nil {
		return err
	}

	return s.q.CreateSession(ctx, repository.CreateSessionParams{
		ID:           sess.ID,
		TokenHash:    sess.TokenHash,
		UserID:       sess.UserID,
		Data:         data,
		Ip:           sess.IP,
		UserAgent:    sess.UserAgent,
		Device:       sess.Device,
		Fingerprint:  sess.Fingerprint,
		CreatedAt:    sess.CreatedAt,
		LastActiveAt: sess.LastActiveAt,
		ExpiresAt:    sess.ExpiresAt,
	})
}

func (s *Store) GetByTokenHash(ctx context.Context, tokenHash string) (*forge.Session, error) {
	row, err := s.q.GetSessionByTokenHash(ctx, tokenHash)
	if err != nil {
		if err == pgx.ErrNoRows {
			return nil, forge.ErrSessionNotFound
		}
		return nil, err
	}

	return rowToSession(row)
}

func (s *Store) Update(ctx context.Context, sess *forge.Session) error {
	data, err := json.Marshal(sess.Data)
	if err != nil {
		return err
	}

	return s.q.UpdateSession(ctx, repository.UpdateSessionParams{
		UserID:       sess.UserID,
		Data:         data,
		Ip:           sess.IP,
		UserAgent:    sess.UserAgent,
		Device:       sess.Device,
		Fingerprint:  sess.Fingerprint,
		LastActiveAt: sess.LastActiveAt,
		ExpiresAt:    sess.ExpiresAt,
		ID:           sess.ID,
	})
}

func (s *Store) Delete(ctx context.Context, id string) error {
	return s.q.DeleteSession(ctx, id)
}

func (s *Store) ListByUserID(ctx context.Context, userID string) ([]*forge.Session, error) {
	rows, err := s.q.ListSessionsByUserID(ctx, &userID)
	if err != nil {
		return nil, err
	}

	sessions := make([]*forge.Session, 0, len(rows))
	for _, row := range rows {
		sess, err := listRowToSession(row)
		if err != nil {
			return nil, err
		}
		sessions = append(sessions, sess)
	}
	return sessions, nil
}

func (s *Store) CountByUserID(ctx context.Context, userID string) (int, error) {
	count, err := s.q.CountSessionsByUserID(ctx, &userID)
	if err != nil {
		return 0, err
	}
	return int(count), nil
}

func (s *Store) DeleteByUserID(ctx context.Context, userID string) error {
	return s.q.DeleteSessionsByUserID(ctx, &userID)
}

func (s *Store) DeleteByUserIDExcept(ctx context.Context, userID, exceptID string) error {
	return s.q.DeleteSessionsByUserIDExcept(ctx, repository.DeleteSessionsByUserIDExceptParams{
		UserID: &userID,
		ID:     exceptID,
	})
}

func (s *Store) DeleteOldestByUserID(ctx context.Context, userID string) error {
	return s.q.DeleteOldestSessionByUserID(ctx, &userID)
}

func (s *Store) Touch(ctx context.Context, id string, lastActiveAt time.Time) error {
	return s.q.TouchSession(ctx, repository.TouchSessionParams{
		LastActiveAt: lastActiveAt,
		ID:           id,
	})
}

func rowToSession(row repository.Session) (*forge.Session, error) {
	var data map[string]any
	if len(row.Data) > 0 {
		if err := json.Unmarshal(row.Data, &data); err != nil {
			return nil, err
		}
	}
	if data == nil {
		data = make(map[string]any)
	}

	return &forge.Session{
		ID:           row.ID,
		TokenHash:    row.TokenHash,
		UserID:       row.UserID,
		Data:         data,
		IP:           row.Ip,
		UserAgent:    row.UserAgent,
		Device:       row.Device,
		Fingerprint:  row.Fingerprint,
		CreatedAt:    row.CreatedAt,
		LastActiveAt: row.LastActiveAt,
		ExpiresAt:    row.ExpiresAt,
	}, nil
}

func listRowToSession(row repository.ListSessionsByUserIDRow) (*forge.Session, error) {
	var data map[string]any
	if len(row.Data) > 0 {
		if err := json.Unmarshal(row.Data, &data); err != nil {
			return nil, err
		}
	}
	if data == nil {
		data = make(map[string]any)
	}

	return &forge.Session{
		ID:           row.ID,
		TokenHash:    row.TokenHash,
		UserID:       row.UserID,
		Data:         data,
		IP:           row.Ip,
		UserAgent:    row.UserAgent,
		Device:       row.Device,
		Fingerprint:  row.Fingerprint,
		CreatedAt:    row.CreatedAt,
		LastActiveAt: row.LastActiveAt,
		ExpiresAt:    row.ExpiresAt,
	}, nil
}
```

### Adapter Notes

- The compile-time check (`var _ forge.SessionStore = (*Store)(nil)`) ensures the adapter stays in sync with the interface.
- `Data` field: `forge.Session` uses `map[string]any`, stored as `JSONB`. The adapter marshals/unmarshals via `json.RawMessage`.
- `UserID` is `*string` in `forge.Session` — maps to nullable `TEXT` in PostgreSQL. sqlc generates this as `*string`.
- `pgx.ErrNoRows` is mapped to `forge.ErrSessionNotFound` in `GetByTokenHash`.
- The `ListByUserID` query returns a separate row type (`ListSessionsByUserIDRow`), so a dedicated conversion function is used.
- All methods that take `userID` as a parameter pass `&userID` because sqlc generates nullable columns as `*string`.
- The `Ip` field name (lowercase) comes from sqlc's default column name mapping of the `ip` column.
