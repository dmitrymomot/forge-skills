# Assets Embed Package

Generate `assets/embed.go` when ANY frontend option (`htmx`, `alpine`, or `tailwind`) is selected.

## Template

```go
package assets

import "embed"

//go:embed static
var StaticFS embed.FS
```

## Usage

Import from main.go:

```go
"{{MODULE_PATH}}/assets"
```

Then use in app options:

```go
forge.WithStaticFiles("/static/", assets.StaticFS, "static")
```

## Notes

- This file is generated **once**, regardless of how many frontend options are selected.
- The `//go:embed static` directive embeds the entire `assets/static/` subtree.
- The embed var is `StaticFS` (exported) so it can be imported by `cmd/main.go`.
- The subdirectory path passed to `WithStaticFiles` is `"static"` (not `"assets/static"`) because the embed.FS root is the `assets/` package directory.
