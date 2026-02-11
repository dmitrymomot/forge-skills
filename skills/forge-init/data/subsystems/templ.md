# templ Subsystem

templ is a type-safe Go HTML templating engine. When selected, it adds tool dependencies to go.mod and creates the template directory structure.

## Tool Dependencies

Added to the `tool (...)` directive in go.mod:

```
github.com/a-h/templ/cmd/templ
github.com/templui/templui/cmd/templui
```

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

None — static file serving is handled by the shared frontend concern (see `subsystem-deps.md` resolution rule #3). Template rendering is done in handlers.

## Shutdown Hook

None.

## Env Vars (dev)

None.

## Env Vars (example)

None.

## Docker Service

None.

## Air Configuration

When templ is selected, the `.air.toml` `include_ext` list should include `"templ"` to trigger hot-reload on template changes:

```toml
include_ext = ["go", "tpl", "tmpl", "html", "templ"]
```

## Documentation Links

- templ: https://templ.guide/
- templui: https://templui.io/docs/how-to-use

## Notes

- templ files (`.templ`) are Go source files — they live in Go packages (e.g., `internal/views/`). The user decides the package layout.
- templ generates Go code from `.templ` files — run `go tool templ generate` before building.
- templui provides pre-built UI components for templ (buttons, forms, modals, etc.).
- The Air config should watch `.templ` files so changes trigger a rebuild.
- Print the documentation links in the Phase 8 summary when templ is enabled.
