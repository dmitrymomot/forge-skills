# Forge Conventions & API Quick Reference

Always-loaded reference for forge-build. These rules apply to ALL generated code.

---

## ID Generation

- Always use `id.New()` from `github.com/dmitrymomot/forge/pkg/id`
- Database column: `VARCHAR(26) PRIMARY KEY`
- Never use `uuid.New()`, `ulid.Make()`, `SERIAL`, `BIGSERIAL`, `IDENTITY`, or `uuid_generate_v4()`
- ID is always the first INSERT parameter (`$1`) — caller generates it

---

## Binding & Validation

Request binding: `c.Bind(&req)` — auto-detects JSON/form based on Content-Type.

Tag separators use **semicolons** (not commas):

```go
type CreatePostRequest struct {
    Title string `json:"title" form:"title" validate:"required;max:255" sanitize:"trim"`
    Body  string `json:"body"  form:"body"  validate:"required"         sanitize:"trim"`
}
```

Common validate tags: `required`, `max:N`, `min:N`, `email`, `url`, `oneof:a;b;c`
Common sanitize tags: `trim`, `lower`, `upper`, `numeric`, `alpha`

---

## Type-Safe Params

```go
id := forge.Param[string](c, "id")            // path param as string
page := forge.QueryDefault[int](c, "page", 1)  // query param with default
limit := forge.QueryDefault[int](c, "limit", 20)
```

---

## Error Handling

- Return errors from handlers — Forge error middleware handles the rest
- `forge.ErrNotFound(msg)` → 404
- `forge.ErrBadRequest(msg)` → 400
- `forge.ErrUnauthorized(msg)` → 401
- `forge.ErrForbidden(msg)` → 403
- `forge.ErrConflict(msg)` → 409
- `forge.ErrInternal(msg)` → 500
- `c.Error(code, message, opts...)` → custom HTTP error (prefer typed constructors above)
- Validation errors: return 422 with field-level errors

---

## Naming Conventions

| Context | Style | Example |
|---|---|---|
| SQL tables/columns | snake_case | `user_profiles`, `created_at` |
| Go types/functions | PascalCase | `UserProfile`, `CreateUser` |
| URLs/routes | kebab-case | `/user-profiles`, `/api/v1` |
| JSON fields | camelCase | `firstName`, `createdAt` |
| Query names (sqlc) | PascalCase | `GetUserByID`, `ListActiveUsers` |

---

## Import Grouping

Always group imports in this order with blank lines between:

```go
import (
    "context"
    "net/http"

    "github.com/dmitrymomot/forge"
    "github.com/dmitrymomot/forge/pkg/id"

    "myapp/internal/repository"
)
```

1. Standard library
2. External packages
3. Internal packages

---

## Context Rules

- `forge.Context` embeds `context.Context` — pass directly to DB calls, HTTP clients
- Session methods (`UserID()`, `IsAuthenticated()`, etc.) are **NOT** goroutine-safe
- Extract values before spawning goroutines: `userID := c.UserID()`

---

## Modern Go (1.25+)

- Use `for range n` instead of `for i := 0; i < n; i++`
- Use `min()`/`max()` builtins
- Use `slices.Contains()` instead of manual loops
- Use `clear()` for maps/slices
- No `v := v` in loop captures (Go 1.22+ fixed loop variables)

---

## Build Verification

```bash
go build -o /dev/null ./...
```

Never leave binary artifacts. Always use `-o /dev/null`.

---

## Forge Context — Response Methods

| Method | Description |
|---|---|
| `c.JSON(code, data)` | JSON response |
| `c.String(code, text)` | Plain text response |
| `c.NoContent(code)` | Empty response (204, etc.) |
| `c.Redirect(code, url)` | Redirect (auto-handles HTMX) |
| `c.Render(code, component)` | Render templ component |
| `c.RenderPartial(code, full, partial)` | Full page or HTMX partial |
| `c.Error(code, message, opts...)` | Custom HTTP error |
| `c.SSE(events)` | Stream server-sent events |

## Forge Context — Data Methods

| Method | Description |
|---|---|
| `c.Bind(&req)` | Bind request body (JSON/form) |
| `c.BindQuery(&req)` | Bind query parameters |
| `c.BindJSON(&req)` | Bind JSON body explicitly |
| `c.Cookie(name)` | Get cookie value |
| `c.SetCookie(name, value, maxAge)` | Set cookie |
| `c.Flash(key, dest)` | Get flash value by key |
| `c.SetFlash(key, value)` | Set flash value by key |
| `c.Logger()` | Get structured logger |
| `c.LogInfo(msg, args...)` | Log info |
| `c.LogError(msg, args...)` | Log error |
| `c.LogWarn(msg, args...)` | Log warning |

## Forge Context — Session Methods

| Method | Description |
|---|---|
| `c.Session()` | Access session store |
| `c.AuthenticateSession(userID)` | Set authenticated user |
| `c.DestroySession()` | Clear session |
| `c.UserID()` | Get authenticated user ID |
| `c.IsAuthenticated()` | Check if logged in |
| `c.Can(permission)` | Check RBAC permission |
| `c.Role()` | Get user role |
| `forge.SessionGet[T](c, key)` | Get typed session value |
| `forge.SessionSet(c, key, value)` | Set typed session value |

## Forge Context — Storage Methods

| Method | Description |
|---|---|
| `c.Upload(field, opts...)` | Upload file from form field |
| `c.UploadFromURL(url, opts...)` | Upload from remote URL |
| `c.Download(key)` | Download file as reader |
| `c.DeleteFile(key)` | Delete file from storage |
| `c.FileURL(key, opts...)` | Generate file URL |
| `c.Storage()` | Direct storage client |

## Forge Context — Job Methods

| Method | Description |
|---|---|
| `c.Enqueue(name, payload, opts...)` | Enqueue background job |
| `c.EnqueueTx(tx, name, payload, opts...)` | Enqueue within DB transaction |

---

## Forge pkg/ Quick Reference

| Package | Import | Purpose |
|---|---|---|
| `pkg/id` | `forge/pkg/id` | ULID generation: `id.New()` |
| `pkg/binder` | `forge/pkg/binder` | Struct binding from requests |
| `pkg/cache` | `forge/pkg/cache` | In-memory TTL cache |
| `pkg/slug` | `forge/pkg/slug` | URL-safe slug generation |
| `pkg/token` | `forge/pkg/token` | Secure random tokens |
| `pkg/sanitizer` | `forge/pkg/sanitizer` | Input sanitization |
| `pkg/secrets` | `forge/pkg/secrets` | Encryption/decryption |
| `pkg/qrcode` | `forge/pkg/qrcode` | QR code generation |
| `pkg/totp` | `forge/pkg/totp` | TOTP 2FA |
| `pkg/dnsverify` | `forge/pkg/dnsverify` | DNS verification |
| `pkg/geolocation` | `forge/pkg/geolocation` | IP geolocation |
| `pkg/fingerprint` | `forge/pkg/fingerprint` | Device fingerprinting |
| `pkg/clientip` | `forge/pkg/clientip` | Client IP extraction |
| `pkg/htmx` | `forge/pkg/htmx` | HTMX request helpers |

---

## Available Middlewares

| Middleware | Usage |
|---|---|
| `middlewares.RequestID()` | Add request ID header |
| `middlewares.Recover()` | Panic recovery |
| `middlewares.CORS(cfg)` | Cross-origin resource sharing |
| `middlewares.JWT(cfg)` | JWT authentication |
| `middlewares.CSRF(cfg)` | CSRF protection |
| `middlewares.RateLimit(cfg)` | Rate limiting |
| `middlewares.AuditLog(cfg)` | Audit logging |
| `middlewares.RequireAuthenticated()` | Require session auth |
| `middlewares.RequirePermission(perm)` | Require specific RBAC permission |
| `middlewares.RequireAnyPermission(perms...)` | Require any of listed permissions |
| `middlewares.I18n(cfg)` | Internationalization |
