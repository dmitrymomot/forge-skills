# Interactive Questions

Ask these questions using the `AskUserQuestion` tool. Adapt based on `$ARGUMENTS` — if an app name is provided, skip Q1's module path question portion about the name.

---

## Q1: Module Path + Run Mode (single AskUserQuestion call with 2 questions)

Combine these into one `AskUserQuestion` call using the `questions` array with 2 items:

**Question 1 — Module Path:**
If `$ARGUMENTS` is provided, use it as the app name and derive a suggested module path.

"What is the Go module path for your project?"

**Options:**
- A. `github.com/<user>/<app-name>` (suggested based on app name)
- B. Custom module path

Default suggestion: derive from app name by lowercasing and replacing spaces/special chars with hyphens.

**Question 2 — Run Mode:**

"How will your app handle domains?"

**Options:**
- A. **Single-domain** (`app.Run()`) — One application, one domain. Simpler setup, most common for micro-SaaS.
- B. **Multi-domain** (`forge.Run()`) — Multiple domain patterns with a fallback app. Use for multi-tenant or white-label setups.

---

## Q2: Subsystems (multi-select)

**Question:** "Which subsystems and frontend tools do you want to enable? Select all that apply."

**Options:**
- A. **PostgreSQL database** — Connection pool, health checks, migrations support
- B. **Redis** — Caching, rate limiting, pub/sub
- C. **Server-side sessions** — Cookie-based sessions with configurable store
- D. **Background jobs** (River) — Async task processing (requires PostgreSQL)
- E. **File storage** (S3-compatible) — Upload/download with RustFS for local dev
- F. **templ** — Type-safe Go HTML templating engine with templui components
- G. **HTMX** — Hypermedia-driven interactions (vendored JS)
- H. **Alpine.js** — Lightweight reactive JS framework (vendored JS)
- I. **TailwindCSS** — Utility-first CSS framework (CLI build)
- J. **Email** (SMTP / Mailpit) — Transactional email with template rendering
- K. **OAuth** (Google/GitHub) — Social login providers
- L. **None** — Minimal app with health checks only

If the user selects `L`, no subsystems are enabled.

---

## Post-Q2: Dependency Resolution

After collecting Q2 answers, resolve dependencies per `data/subsystem-deps.md`:

1. Check if `jobs` is selected without `db` → auto-add `db`, inform user
2. Check if `oauth` is selected without `sessions` → recommend adding `sessions`
3. Check if ANY of `htmx`, `alpine`, `tailwind` is selected → note shared assets directory will be created
4. Display final resolved subsystem list
5. Ask user to confirm before proceeding
