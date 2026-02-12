# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

A Claude Code plugin (skills + agents) for building applications with the [Forge framework](https://github.com/dmitrymomot/forge) — a Go micro-SaaS framework. Skills in this repo provide Forge-aware code generation, architecture guidance, and workflow automation.

## Forge Framework Quick Reference

Module: `github.com/dmitrymomot/forge`

**Core types** (all re-exported from `internal/` via `forge.go`):

- `forge.App`, `forge.Context`, `forge.Router`, `forge.Handler`, `forge.HandlerFunc`, `forge.Middleware`
- `forge.New()` creates an app, `forge.Run()` starts the server
- `forge.Context` embeds `context.Context` — pass directly to DB calls, HTTP clients, etc.

**Handler pattern:**

```go
type MyHandler struct{ repo *repository.Queries }

func (h *MyHandler) Routes(r forge.Router) {
    r.GET("/items", h.list)
    r.POST("/items", h.create)
}

func (h *MyHandler) list(c forge.Context) error {
    return c.JSON(200, items)
}
```

**Key subsystems** (all optional, enabled via `forge.With*` options):

- Sessions: `WithSession()`, then `c.AuthenticateSession()`, `c.IsAuthenticated()`, `c.UserID()`
- RBAC: `WithRoles()`, then `c.Can("permission")`, `c.Role()`
- Jobs: `WithJobs(pool, job.Config{}, ...)` (River-based), register with `job.WithTask`/`job.WithScheduledTask`
- Storage: `WithStorage()` (S3-compatible), then `c.Upload()`, `c.FileURL()` (returns `string, error`)
- HTMX: `c.IsHTMX()`, `c.RenderPartial(fullPage, partial)`

**Utility packages** in `pkg/`: binder, cache, clientip, cookie, db, dnsverify, fingerprint, geolocation, hostrouter, htmx, i18n, id, job, jwt, logger, mailer, oauth, qrcode, ratelimit, redis, sanitizer, secrets, slug, storage, token, totp

**Middlewares** in `middlewares/`: audit, auth, cors, csrf, errors, i18n, jwt, ratelimit, rbac, recover, requestid

**Data binding/validation** uses semicolon-separated tags: `validate:"required;max:100"` / `sanitize:"trim;numeric"`

**Type-safe params**: `forge.Param[int64](c, "id")`, `forge.QueryDefault[int](c, "page", 1)`

## Forge API Gotchas (Common Mistakes in Data Files)

- Route params: `{id}` not `:id` — Forge uses chi-style routing
- `c.Bind(&req)` returns `(ValidationErrors, error)` — check both
- `c.AuthenticateSession(userID)` returns `error` — must check it
- `forge.SessionGet[T](c, key)` / `forge.SessionSet(c, key, value)` — standalone generics, not methods
- `c.FileURL(key, opts...)` returns `(string, error)` — not just string
- SSE constructors require event name: `forge.SSEJSON(event, data)`, `forge.SSEString(event, data)`
- `storage.NotEmpty()` not `storage.FileNotEmpty()`
- `c.SetCookie(name, value, maxAge)` — three args, not a cookie struct
- `c.Flash(key, dest)` / `c.SetFlash(key, value)` — key-value based
- River default retry: 25 attempts, not 3. Priority: lower number = higher
- Verify data files against source at `../forge` when accuracy matters

## Forge Design Rules (Skills Must Enforce)

- No reflection, no service containers, no magic
- Packages receive values via parameters, not context
- All IDs via `pkg/id/` — never `uuid.New()` or `ulid.Make()`
- Modern Go 1.25+: `for range n`, `min()`/`max()`, `clear()`, `slices.Contains()`
- Validator/sanitizer tags use semicolons as separators, colons for params
- `go build -o /dev/null ./...` — never leave binary artifacts
- `requestContext` session methods are NOT goroutine-safe
- `recover()` returns `*runtime.PanicNilError` in Go 1.21+
- No `v := v` captures (Go 1.22+ loop variables)

## Skills Structure

Each skill lives in its own directory with a `SKILL.md` file:

```
skills/
  skill-name/
    SKILL.md          # Frontmatter + prompt body
```

### SKILL.md Frontmatter

```yaml
---
name: skill-name # Unique identifier, used as /skill-name command
description: One-line description # Shown in skill list, used for auto-matching
argument-hint: <hint> # Optional placeholder for $ARGUMENTS
context: fork # "fork" = fresh context, "current" = inherits conversation
agent: agent-name # Optional: delegates to a specific agent
allowed-tools: Read, Grep, Bash # Optional: restrict available tools
---
```

- `$ARGUMENTS` in the body is replaced with user's input after the slash command
- Skills that delegate to agents use `agent:` to specify which agent handles execution
- `context: fork` is preferred for isolated tasks; omit or use `current` when conversation context is needed

### Agent Definitions

Agents live in `.claude/agents/` (at user level `~/.claude/agents/` or project level):

```yaml
---
name: agent-name
description: What this agent does
model: sonnet | opus # sonnet for fast code tasks, opus for architecture
tools: Read, Grep, Glob, Edit, Write, Bash
permissionMode: plan # Optional: requires plan approval before acting
---
```

## Adding a New Forge Skill

1. Create `skills/<name>/SKILL.md` with proper frontmatter
2. If the skill needs a dedicated agent, create the agent definition too
3. Skills should embed Forge conventions (ID generation, error patterns, binding tags, etc.)
4. Code-generating skills must verify compilation with `go build -o /dev/null`
5. Skills that use third-party APIs must verify them with `go doc` first — never guess

## Editing Gotchas

- `skills/forge-init/data/base-app.md` has near-identical single-domain and multi-domain templates — include the `## main.go —` heading in edit context to avoid duplicate matches
- Plugin version lives in `.claude-plugin/marketplace.json` — appears twice (metadata + plugin), bump both with `replace_all`

## Relationship to Global Skills

The user has global skills at `~/.claude/skills/` covering Go development (`go-code`, `go-test`, `go-doc`, `go-review`, `go-comments`, `go-fix`, `go-readme`), architecture (`system-arch`, `security-arch`, `data-arch`, `ops-arch`), product (`prd`, `product-strategy`, `idea-validate`, `tech-research`), and git workflows (`git-commit`, `git-pr`, `git-release`, `git-clean`, `git-gh`). Skills in this repo should complement those by adding **Forge-specific** knowledge — not duplicate generic Go or architecture guidance.
