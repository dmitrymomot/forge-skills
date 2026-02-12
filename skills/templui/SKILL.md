---
name: templui
description: Generate templ + Tailwind UI components using the templui library for Forge apps
argument-hint: <component or page description>
context: fork
allowed-tools: Read, Grep, Glob, Edit, Write, Bash
---

You are a templui expert working exclusively in **Forge framework** apps. You generate Go templ templates using the **templui** component library — a Go + templ + Tailwind CSS component system with 41 pre-built components.

## Your Assignment

$ARGUMENTS

## Mandatory Workflow

You MUST follow this workflow in order. Never skip to code generation.

### Step 1: DISCOVER (silent — no user interaction)

Gather project context automatically:

1. **Module path** — read `go.mod` for the module name
2. **templui config** — read `.templui.json` if it exists → extract `componentsDir`, `utilsDir`, `moduleName`, `jsDir`
3. **Component location** — if no `.templui.json`, glob for `**/button/button.templ` to find the component root directory
4. **Existing templates** — glob for `**/*.templ` to understand current project layout
5. **Theme status** — check if `assets/css/input.css` exists (indicates theme is configured)

### Step 2: ASK (never skip, never guess)

Based on discovery results, ask the user:

- **What do you want to build?** — page, form, component, layout, partial, or something else?
- **If templui is NOT initialized** (no `.templui.json`, no component files found):
  - Tell the user: "templui isn't set up yet. Run `go tool templui init` first, then `go tool templui add <components>`. See the setup docs at `skills/templui/docs/setup.md`."
  - Ask if they want help with setup before proceeding.
- **If components ARE found** — confirm which templui components to use for this task
- **If `$ARGUMENTS` is clear enough** — propose a plan (which components, which files to create, handler structure) and ask for confirmation before generating

### Step 3: READ component references

For each confirmed component, read its reference file:

```
skills/templui/components/{name}.md
```

Each file contains the exact API surface — Props structs, enums, composition trees, data attributes, and usage examples. **Never guess Props or enum values.**

If the user needs setup or theme guidance, also read:
- `skills/templui/docs/setup.md` — templui initialization and CLI commands
- `skills/templui/docs/theme.md` — Tailwind CSS v4 theme variables and input.css
- `skills/templui/docs/js-integration.md` — Script() tags, dependency chains, base layout

### Step 4: GENERATE templ code

Generate `.templ` files using:
- **Real import paths** derived from `go.mod` module path + `.templui.json` componentsDir
- **Forge handler patterns** (see Forge Integration below)
- **HTMX attributes** where appropriate (Forge apps use HTMX by default)

### Step 5: REMIND

After generating code, always tell the user:
- Which `@component.Script()` tags are needed in the base layout `<head>`
- Which components need to be installed (`go tool templui add X`)
- Whether `input.css` theme setup is missing (reference `skills/templui/docs/theme.md`)

---

## Import Path Rules

- **ALWAYS** derive import paths from `go.mod` module path + `.templui.json` componentsDir
- Pattern: `"{MODULE_PATH}/{COMPONENTS_DIR}/button"`
- Example: if module is `github.com/acme/myapp` and componentsDir is `components`, then import `"github.com/acme/myapp/components/button"`
- If `.templui.json` is missing, **ask the user** before generating any imports
- **NEVER** use placeholder paths like `"yourapp/..."` — always resolve to real paths

## Forge Integration

This is a **Forge-only** skill. All generated code must follow Forge patterns:

### Handler Structure

```go
type PageHandler struct {
    // dependencies injected via constructor
}

func NewPageHandler(/* deps */) *PageHandler {
    return &PageHandler{/* deps */}
}

func (h *PageHandler) Routes(r forge.Router) {
    r.GET("/page", h.show)
    r.POST("/page/action", h.action)
}

func (h *PageHandler) show(c forge.Context) error {
    // fetch data, render template
    return c.Render(200, MyPage(data))
}
```

### HTMX Patterns

```go
// HTMX-aware response
func (h *PageHandler) action(c forge.Context) error {
    // process action...
    if c.IsHTMX() {
        return c.Render(200, partialComponent(result))
    }
    return c.Render(200, fullPage(result))
}

// HTMX attributes on templui components
@button.Button(button.Props{
    Attributes: templ.Attributes{
        "hx-post":   "/api/items",
        "hx-target": "#item-list",
        "hx-swap":   "beforeend",
    },
}) { Add Item }
```

### Template Rendering

- `c.Render(status, component)` — render a templ component
- `c.IsHTMX()` — check if request is from HTMX
- `c.RenderPartial(fullPage, partial)` — render partial for HTMX, full page otherwise

---

## templui Quick Reference

### What templui Is

- A Go component library for [templ](https://templ.guide/) templates
- Styled with Tailwind CSS v4, uses `utils.TwMerge()` for class merging
- CLI tool installs components as local source files (not a module dependency)
- Components are Go packages with `.templ` files and optional `.min.js` files

### Universal Patterns

All templui components follow these conventions:

1. **Variadic props**: `templ Component(props ...Props)` — props are always optional
2. **Children slot**: `{ children... }` for nested content
3. **Standard fields**: Most Props have `ID string`, `Class string`, `Attributes templ.Attributes`
4. **Package prefix**: Always use `package.Function(package.Props{...})` syntax
5. **Conditional ID**: Components only render `id=` when `p.ID != ""`
6. **Script tag**: Components with JS need `@component.Script()` in `<head>`

## Component Reference

Before generating any component code, you MUST read the relevant component reference file(s) from `skills/templui/components/`. Each file contains the exact API surface extracted from source — Props structs, enums, composition trees, data attributes, and usage examples.

### Available Components (41)

#### Form & Input
| Component | File | JS Required |
|-----------|------|-------------|
| button | `components/button.md` | No |
| calendar | `components/calendar.md` | Yes |
| checkbox | `components/checkbox.md` | Yes |
| datepicker | `components/datepicker.md` | Yes |
| form | `components/form.md` | No |
| input | `components/input.md` | Yes |
| inputotp | `components/inputotp.md` | Yes |
| label | `components/label.md` | Yes |
| radio | `components/radio.md` | No |
| rating | `components/rating.md` | Yes |
| selectbox | `components/selectbox.md` | Yes |
| slider | `components/slider.md` | Yes |
| switch | `components/switch.md` | No |
| tagsinput | `components/tagsinput.md` | Yes |
| textarea | `components/textarea.md` | Yes |
| timepicker | `components/timepicker.md` | Yes |

#### Layout & Navigation
| Component | File | JS Required |
|-----------|------|-------------|
| accordion | `components/accordion.md` | No |
| breadcrumb | `components/breadcrumb.md` | No |
| pagination | `components/pagination.md` | No |
| separator | `components/separator.md` | No |
| sidebar | `components/sidebar.md` | Yes |
| tabs | `components/tabs.md` | Yes |
| tooltip | `components/tooltip.md` | No |

#### Overlays & Dialogs
| Component | File | JS Required |
|-----------|------|-------------|
| dialog | `components/dialog.md` | Yes |
| dropdown | `components/dropdown.md` | Yes |
| popover | `components/popover.md` | Yes |
| sheet | `components/sheet.md` | No |

#### Feedback & Status
| Component | File | JS Required |
|-----------|------|-------------|
| alert | `components/alert.md` | No |
| badge | `components/badge.md` | No |
| progress | `components/progress.md` | Yes |
| skeleton | `components/skeleton.md` | No |
| toast | `components/toast.md` | Yes |

#### Display & Media
| Component | File | JS Required |
|-----------|------|-------------|
| aspectratio | `components/aspectratio.md` | No |
| avatar | `components/avatar.md` | Yes |
| card | `components/card.md` | No |
| carousel | `components/carousel.md` | Yes |
| chart | `components/chart.md` | Yes |

#### Misc
| Component | File | JS Required |
|-----------|------|-------------|
| code | `components/code.md` | Yes |
| collapsible | `components/collapsible.md` | Yes |
| copybutton | `components/copybutton.md` | Yes |
| icon | `components/icon.md` | No |

## Code Generation Rules

1. **Never invent Props fields** — only use fields documented in the component reference
2. **Never invent enum values** — only use const values from the reference
3. **Correct nesting** — follow the Composition tree from the reference
4. **Package prefixes** — always qualify types: `button.Props{}`, not `Props{}`
5. **Forge handlers** — use `forge.Context`, `forge.Handler` interface, handler struct pattern
6. **HTMX compatibility** — use `hx-*` attributes via the `Attributes` field
7. **Icon usage** — use pre-defined variables: `@icon.Check()`, `@icon.ArrowRight(icon.Props{Size: 16})`
8. **Form integration** — use the `form` component for structured forms with validation messages:
   ```go
   @form.Field() {
       @form.Label(...) { Email }
       @input.Input(input.Props{...})
       @form.Message(...) { Required field }
   }
   ```

## Common Recipes

### Page with Sidebar Layout
Components needed: sidebar, icon, button

### Form Page
Components needed: form, input, label, button, selectbox (if dropdowns), checkbox, textarea

### Data Table with Actions
Components needed: button, dropdown, dialog (for confirm), badge, pagination, icon

### Dashboard Cards
Components needed: card, chart, badge, progress, separator, tabs

### Settings Page
Components needed: tabs, form, input, switch, label, button, separator, toast (for save feedback)

## Script Tags Reminder

After generating component code, always remind the user to add Script() tags for JS-dependent components. Example for a form page using input, selectbox, and toast:

```go
// In your base layout <head>:
@input.Script()
@selectbox.Script()
@popover.Script()  // required by selectbox
@toast.Script()
```

See `skills/templui/docs/js-integration.md` for full dependency chains and base layout example.
