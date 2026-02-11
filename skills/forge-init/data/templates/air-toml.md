# .air.toml — Hot Reload Configuration

Generate this file at the project root for development hot-reloading with [Air](https://github.com/air-verse/air).

## Template

```toml
root = "."
tmp_dir = "tmp"

[build]
  bin = "./tmp/main"
  cmd = "go build -o ./tmp/main ./cmd/..."
  delay = 1000
  exclude_dir = ["assets", "tmp", "vendor", "node_modules", ".git"]
  exclude_file = []
  exclude_regex = ["_test.go"]
  exclude_unchanged = false
  follow_symlink = false
  full_bin = ""
  include_dir = []
  include_ext = ["go", "tpl", "tmpl", "html"]
  include_file = []
  kill_delay = "0s"
  log = "build-errors.log"
  poll = false
  poll_interval = 0
  rerun = false
  rerun_delay = 500
  send_interrupt = false
  stop_on_error = false

[color]
  app = ""
  build = "yellow"
  main = "magenta"
  runner = "green"
  watcher = "cyan"

[log]
  main_only = false
  time = false

[misc]
  clean_on_exit = true

[screen]
  clear_on_rebuild = false
  keep_scroll = true
```

## Conditional: templ support

When `templ` is selected, add `"templ"` to the `include_ext` list so Air watches `.templ` files:

```toml
  include_ext = ["go", "tpl", "tmpl", "html", "templ"]
```

Also change the build command to run templ generate before building:

```toml
  cmd = "go tool templ generate && go build -o ./tmp/main ./cmd/..."
```

## Notes

- The `cmd` builds from `./cmd/...` matching the project structure.
- `include_ext` includes `html` and `tmpl` for template changes.
- When templ is enabled, `.templ` files are also watched for hot-reload.
- Air runs via `go tool air` from the go.mod tool directive.
