# HTMX Subsystem

HTMX is a vendored JS library downloaded into `assets/static/js/`. It does NOT add any Go code, config fields, or embed directives on its own — the shared frontend concern (static embed + `forge.WithStaticFiles`) is handled once for all frontend options.

## Imports (config.go)

None.

## Imports (main.go)

None.

## Config Field

None.

## Init Code

None.

## Health Check

None.

## App Option

None — static file serving is handled by the shared frontend concern (see `subsystem-deps.md` resolution rule #3).

## Shutdown Hook

None.

## Env Vars (dev)

None.

## Env Vars (example)

None.

## Docker Service

None.

## Download Commands

Download HTMX into the vendored assets directory:

```bash
curl -sL https://unpkg.com/htmx.org/dist/htmx.min.js -o assets/static/js/htmx.min.js
```

These commands are included in the `assets:download` Taskfile task.

## Notes

- HTMX is vendored (not loaded from CDN) to avoid external dependencies.
- Use `c.IsHTMX()` to detect HTMX requests in handlers.
- Use `c.RenderPartial(fullPage, partial)` to serve full page or partial based on request type.
