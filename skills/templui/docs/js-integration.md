# JavaScript Integration for templui in Forge Apps

templui components that use JavaScript ship `.min.js` files that get installed to your `jsDir` (configured in `.templui.json`). This guide covers which components need JS, how to include them, and dependency chains.

## Which Components Need JavaScript

24 of 41 templui components require JavaScript. Each exposes a `Script()` templ function that must be called in your base layout's `<head>`.

| Component   | Script() Required |
| ----------- | :---------------: |
| avatar      |        Yes        |
| calendar    |        Yes        |
| carousel    |        Yes        |
| chart       |        Yes        |
| checkbox    |        Yes        |
| code        |        Yes        |
| collapsible |        Yes        |
| copybutton  |        Yes        |
| datepicker  |        Yes        |
| dialog      |        Yes        |
| dropdown    |        Yes        |
| input       |        Yes        |
| inputotp    |        Yes        |
| label       |        Yes        |
| popover     |        Yes        |
| progress    |        Yes        |
| rating      |        Yes        |
| selectbox   |        Yes        |
| slider      |        Yes        |
| tabs        |        Yes        |
| tagsinput   |        Yes        |
| textarea    |        Yes        |
| timepicker  |        Yes        |
| toast       |        Yes        |

Components that do **NOT** need JS: accordion, alert, aspectratio, badge, breadcrumb, button, card, form, icon, pagination, radio, separator, sheet, skeleton, switch, tooltip.

## Script() Pattern

Every JS-requiring component has a `Script()` function. Call it once in your base layout `<head>`:

```go
@input.Script()
@toast.Script()
@dialog.Script()
```

**Important:** Call `Script()` only once per component, in the base layout — not in every page that uses the component.

## Dependency Chains

Some components internally use other components and need their Script() tags too. If you only include the parent's Script(), the child's JS won't load.

### Components with Dependencies

| Component  | Also Needs                                    |
| ---------- | --------------------------------------------- |
| selectbox  | `@popover.Script()`                           |
| tagsinput  | `@popover.Script()`                           |
| datepicker | `@popover.Script()`                           |
| dropdown   | `@popover.Script()`                           |
| tooltip    | `@popover.Script()` (uses popover internally) |
| sidebar    | `@sheet.Script()` → `@dialog.Script()`        |

### Example: Form with selectbox

```go
// In base layout <head>:
@input.Script()
@label.Script()
@selectbox.Script()
@popover.Script()   // required by selectbox
@textarea.Script()
@toast.Script()      // for save feedback
```

### Example: Dashboard with sidebar

```go
// In base layout <head>:
@sidebar.Script()
@dialog.Script()     // required by sheet, which sidebar uses
@tabs.Script()
@chart.Script()
@progress.Script()
@toast.Script()
```

## Asset Serving

JS files are automatically handled by the forge-init setup:

- **templui** installs `.min.js` files to `assets/static/js/` (per `.templui.json` `jsDir`)
- **forge** serves them via `forge.WithStaticFiles("/static/", ...)` configured in `main.go`
- **Script()** renders a `<script>` tag with the correct public path (per `.templui.json` `jsPublicPath`)

No additional asset configuration is needed.

## Base Layout Example

Complete base layout with common Script() tags for a typical Forge app:

```go
package layouts

import (
    "{MODULE_PATH}/{COMPONENTS_DIR}/dialog"
    "{MODULE_PATH}/{COMPONENTS_DIR}/dropdown"
    "{MODULE_PATH}/{COMPONENTS_DIR}/input"
    "{MODULE_PATH}/{COMPONENTS_DIR}/label"
    "{MODULE_PATH}/{COMPONENTS_DIR}/popover"
    "{MODULE_PATH}/{COMPONENTS_DIR}/selectbox"
    "{MODULE_PATH}/{COMPONENTS_DIR}/sidebar"
    "{MODULE_PATH}/{COMPONENTS_DIR}/textarea"
    "{MODULE_PATH}/{COMPONENTS_DIR}/toast"
)

templ Base(title string) {
    <!DOCTYPE html>
    <html lang="en">
        <head>
            <meta charset="UTF-8"/>
            <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
            <title>{ title }</title>

            // Theme detection (before CSS to prevent flash)
            <script>
                (function() {
                    const saved = localStorage.getItem('theme');
                    if (saved === 'dark' || (!saved && window.matchMedia('(prefers-color-scheme: dark)').matches)) {
                        document.documentElement.classList.add('dark');
                    }
                })();
            </script>

            // Stylesheets
            <link rel="stylesheet" href="/static/css/output.css"/>

            // Core JS libraries (vendored by forge-init)
            <script src="/static/js/htmx.min.js"></script>
            <script src="/static/js/alpine.min.js" defer></script>

            // templui component scripts
            // Only include Script() for components you actually use.
            // Check the dependency chain table above.
            @dialog.Script()
            @dropdown.Script()
            @input.Script()
            @label.Script()
            @popover.Script()
            @selectbox.Script()
            @sidebar.Script()
            @textarea.Script()
            @toast.Script()
        </head>
        <body class="min-h-screen bg-background text-foreground antialiased">
            { children... }

            // Toast container (if using toast notifications)
            @toast.Toaster()
        </body>
    </html>
}
```

**Note:** Replace `{MODULE_PATH}` and `{COMPONENTS_DIR}` with your actual values from `go.mod` and `.templui.json`. For example: `"github.com/acme/myapp/templates/templui/components/toast"`.

## Adding New Components

When you install a new component that requires JS:

1. Run `go tool templui add <component>`
2. Check if it has JS: look for `JS Required: Yes` in the component reference
3. Add `@component.Script()` to your base layout `<head>`
4. Check the dependency chain table — add any required dependency scripts too
5. Run `go tool templ generate` to regenerate Go code
