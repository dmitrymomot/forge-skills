# Alpine.js Subsystem

Alpine.js is a vendored JS library downloaded into `assets/static/js/`. It does NOT add any Go code, config fields, or embed directives on its own — the shared frontend concern (static embed + `forge.WithStaticFiles`) is handled once for all frontend options.

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

Download Alpine.js into the vendored assets directory:

```bash
curl -sL https://unpkg.com/alpinejs/dist/cdn.min.js -o assets/static/js/alpine.min.js
```

These commands are included in the `assets-download` justfile recipe.

## Notes

- Alpine.js is vendored (not loaded from CDN) to avoid external dependencies.
- Include via `<script defer src="/static/js/alpine.min.js"></script>` in your HTML templates.
- Alpine.js provides reactive behavior with `x-data`, `x-bind`, `x-on`, etc. directly in HTML markup.
