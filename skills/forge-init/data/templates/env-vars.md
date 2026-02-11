# Environment Variables

Generate two files: `.env` (dev-ready, gitignored) and `.env.example` (committed, placeholder values).

---

## Base (always included)

### .env (dev values)

```env
# App
APP_BASE_DOMAIN=localhost
APP_REQUEST_TIMEOUT=30s

# Server
RUN_ADDRESS=:8080
RUN_SHUTDOWN_TIMEOUT=10s

# Cookie
COOKIE_SECRET=dev-secret-change-me-must-be-at-least-32-bytes!!
COOKIE_DOMAIN=localhost
COOKIE_PATH=/
COOKIE_SAME_SITE=lax
COOKIE_SECURE=false
COOKIE_HTTP_ONLY=true

# CORS
CORS_ALLOWED_ORIGINS=http://localhost:8080
```

### .env.example (placeholder values)

```env
# App
APP_BASE_DOMAIN=yourdomain.com
APP_REQUEST_TIMEOUT=30s

# Server
RUN_ADDRESS=:8080
RUN_SHUTDOWN_TIMEOUT=10s

# Cookie
COOKIE_SECRET=<generate-a-32+-byte-secret>
COOKIE_DOMAIN=yourdomain.com
COOKIE_PATH=/
COOKIE_SAME_SITE=lax
COOKIE_SECURE=true
COOKIE_HTTP_ONLY=true

# CORS
CORS_ALLOWED_ORIGINS=https://yourdomain.com
```

---

## Per-Subsystem Sections

For each enabled subsystem, append the corresponding env vars from the subsystem's `data/subsystems/*.md` file. Each subsystem has both "Env Vars (dev)" and "Env Vars (example)" sections.

### Assembly Order

Append sections in this order, with a blank line and comment header between each:

1. Base (always)
2. DB (`# Database`)
3. Redis (`# Redis`)
4. Jobs (`# Jobs`)
5. Storage (`# Storage`)
6. Mailer (`# Email`)
7. OAuth (`# OAuth`)

Sessions env vars are already covered by the base Cookie section. When sessions is enabled, the cookie values are already present in the base section.

HTMX has no env vars.

### Example .env (with db + redis)

```env
# App
APP_BASE_DOMAIN=localhost
APP_REQUEST_TIMEOUT=30s

# Server
RUN_ADDRESS=:8080
RUN_SHUTDOWN_TIMEOUT=10s

# Cookie
COOKIE_SECRET=dev-secret-change-me-must-be-at-least-32-bytes!!
COOKIE_DOMAIN=localhost
COOKIE_PATH=/
COOKIE_SAME_SITE=lax
COOKIE_SECURE=false
COOKIE_HTTP_ONLY=true

# Database
DB_URL=postgres://postgres:postgres@localhost:5432/myapp?sslmode=disable

# Redis
REDIS_URL=redis://localhost:6379/0
```

---

## Rules

1. Replace `{{APP_NAME}}` with the actual app name in all values (e.g., DB name, bucket name).
2. Dev values in `.env` MUST match the credentials and ports in `docker-compose.yml`.
3. `.env` is gitignored — safe for local secrets.
4. `.env.example` is committed — shows required vars with safe placeholder values.
5. Both files have identical keys, only values differ.
