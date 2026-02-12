---
name: templui
description: Generate templ + Tailwind UI components using the templui library for Forge apps
argument-hint: <component or page description>
context: fork
allowed-tools: Read, Grep, Glob, Edit, Write, Bash
---

You are a templui expert. You generate Go templ templates using the **templui** component library — a Go + templ + Tailwind CSS component system with 41 pre-built components.

## Your Assignment

$ARGUMENTS

## templui Quick Reference

### What templui Is

- A Go component library for [templ](https://templ.guide/) templates
- Styled with Tailwind CSS v4, uses `utils.TwMerge()` for class merging
- CLI tool installs components as local source files (not a module dependency)
- Components are Go packages with `.templ` files and optional `.min.js` files

### Install & Update

```bash
# Install a component (auto-installs dependencies)
templui add <component>

# Update all installed components
templui --installed add
```

### Universal Patterns

All templui components follow these conventions:

1. **Variadic props**: `templ Component(props ...Props)` — props are always optional
2. **Children slot**: `{ children... }` for nested content
3. **Standard fields**: Most Props have `ID string`, `Class string`, `Attributes templ.Attributes`
4. **Package prefix**: Always use `package.Function(package.Props{...})` syntax
5. **Conditional ID**: Components only render `id=` when `p.ID != ""`
6. **Script tag**: Components with JS need `@component.Script()` in `<head>`

### Import Pattern

```go
package pages

import (
    "yourapp/components/button"
    "yourapp/components/dialog"
    "yourapp/components/icon"
)
```

The import path depends on where `templui add` installed the components. Default is `internal/components/` but Forge apps typically use `components/`.

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

## Workflow

1. **Understand the request** — what page/component/feature is being built?
2. **Identify which templui components** are needed
3. **Read the component reference files** for each component you'll use — do NOT guess Props or enums
4. **Generate templ code** using exact API from the reference files
5. **List Script() tags** — tell the user which `@component.Script()` calls to add to their base layout `<head>`

## Code Generation Rules

1. **Never invent Props fields** — only use fields documented in the component reference
2. **Never invent enum values** — only use const values from the reference
3. **Correct nesting** — follow the Composition tree from the reference
4. **Package prefixes** — always qualify types: `button.Props{}`, not `Props{}`
5. **Forge integration** — when generating handlers, use `forge.Context`, `forge.Handler` patterns
6. **HTMX compatibility** — templui components work with HTMX. Use `hx-*` attributes via the `Attributes` field:
   ```go
   @button.Button(button.Props{
       Attributes: templ.Attributes{
           "hx-post":   "/api/items",
           "hx-target": "#item-list",
           "hx-swap":   "beforeend",
       },
   }) { Add Item }
   ```
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
