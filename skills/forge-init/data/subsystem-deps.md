# Subsystem Dependencies

## Dependency Graph

| Subsystem    | Key          | Requires | Auto-adds | Notes                              |
|--------------|--------------|----------|-----------|------------------------------------|
| PostgreSQL   | db           | —        | —         |                                    |
| Redis        | redis        | —        | —         |                                    |
| Sessions     | sessions     | —        | —         | Cookie config always in base       |
| Jobs         | jobs         | db       | db        | River-based, needs pgxpool         |
| Storage      | storage      | —        | —         | S3-compatible                      |
| templ        | templ        | —        | —         | Go templating engine               |
| HTMX         | htmx         | —        | —         | JS library, vendored to assets     |
| Alpine.js    | alpine       | —        | —         | JS library, vendored to assets     |
| TailwindCSS  | tailwind     | —        | —         | CSS framework, vendored to assets  |
| Mailer       | mailer       | —        | —         | SMTP-based (Mailpit in dev)        |
| OAuth        | oauth        | —        | —         | Sessions strongly recommended      |

## Resolution Rules

1. If `jobs` is selected and `db` is NOT selected, auto-add `db` and inform the user: "Background jobs require PostgreSQL — adding database support automatically."
2. If `oauth` is selected and `sessions` is NOT selected, suggest adding sessions: "OAuth works best with server-side sessions for state management. Add sessions? (recommended)"
3. **Frontend shared assets rule:** If ANY of `htmx`, `alpine`, or `tailwind` is selected, the project gets a shared `assets/` directory tree with an `assets/embed.go` package (see `templates/assets-embed.md`). Import the package in main.go as `"{{MODULE_PATH}}/assets"` and use `forge.WithStaticFiles("/static/", assets.StaticFS, "static")`. These are NOT duplicated per frontend option — they are added once.
4. After resolution, display the final list and ask for confirmation before proceeding.

## Env Prefix Map

| Subsystem  | Config Type              | Env Prefix        |
|------------|--------------------------|-------------------|
| base       | `forge.AppConfig`        | `APP_`            |
| base       | `forge.RunConfig`        | `RUN_`            |
| base       | `cookie.Config`          | `COOKIE_`         |
| base       | `middlewares.CORSConfig` | `CORS_`           |
| db         | `db.Config`              | `DB_`             |
| redis      | `forgeredis.Config`      | `REDIS_`          |
| jobs       | `job.Config`             | `JOB_`            |
| storage    | `storage.Config`         | `STORAGE_`        |
| mailer     | `mailer.Config`          | `MAILER_`         |
| mailer     | `smtp.Config`            | `SMTP_`           |
| oauth      | `oauth.GoogleConfig`     | `GOOGLE_OAUTH_`   |
| oauth      | `oauth.GitHubConfig`     | `GITHUB_OAUTH_`   |
