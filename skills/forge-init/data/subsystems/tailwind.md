# TailwindCSS Subsystem

TailwindCSS is a vendored JS library (browser CDN build) downloaded into `assets/static/js/`. It does NOT add any Go code, config fields, or embed directives on its own — the shared frontend concern (static embed + `forge.WithStaticFiles`) is handled once for all frontend options.

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

Download TailwindCSS browser build into the vendored assets directory:

```bash
curl -sL https://unpkg.com/@tailwindcss/browser@4/dist/cdn.min.js -o assets/static/js/tailwind.min.js
```

These commands are included in the `assets:download` Taskfile task.

## Notes

- TailwindCSS browser build is vendored (not loaded from CDN) to avoid external dependencies.
- Include via `<script src="/static/js/tailwind.min.js"></script>` in your HTML templates.
- This uses the TailwindCSS v4 browser CDN build which processes utility classes at runtime — suitable for development and small projects.
- For production, consider switching to the TailwindCSS CLI build process for optimized output.
