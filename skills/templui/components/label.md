# Label

> Accessible form label with automatic disabled-state styling that reacts to the linked input's `disabled` attribute.

## Install

```bash
templui add label
```

Dependencies auto-installed: none

Add to your base layout `<head>`: `@label.Script()`

## Update

```bash
templui add label
```

To update all: `templui --installed add`

## API

### Enums

No enums.

### Props

```go
type Props struct {
	ID         string
	Class      string
	Attributes templ.Attributes
	For        string
	Error      string
}
```

| Prop | Type | Default | Description |
| ---- | ---- | ------- | ----------- |
| ID | `string` | `""` | HTML `id` attribute for the `<label>`. |
| Class | `string` | `""` | Additional CSS classes merged via `utils.TwMerge`. |
| Attributes | `templ.Attributes` | `nil` | Arbitrary HTML attributes spread onto the `<label>`. |
| For | `string` | `""` | HTML `for` attribute linking the label to an input element by ID. |
| Error | `string` | `""` | When non-empty, applies `text-destructive` styling to the label. |

## Components

| Function | Props | Description |
| -------- | ----- | ----------- |
| `label.Label(props ...Props)` | `label.Props` | Renders a `<label>` element. Accepts children via `{ children... }`. |

## Composition

Flat component -- no sub-components.

## Dependencies

None -- only uses `utils` package.

## Context

No context passing.

## Data Attributes

| Attribute | Set On | Value | Description |
| --------- | ------ | ----- | ----------- |
| `data-tui-label-disabled-style` | `<label>` | `"opacity-50 cursor-not-allowed"` | Space-separated CSS classes applied by JavaScript when the linked input (via `for`) is disabled. Classes are removed when the input becomes enabled again. |

## Usage

```templ
package pages

import (
	"github.com/templui/templui/internal/components/label"
	"github.com/templui/templui/internal/components/input"
)

templ ProfileForm() {
	<form method="post" action="/profile">
		@label.Label(label.Props{For: "username"}) {
			Username
		}
		@input.Input(input.Props{
			ID:          "username",
			Name:        "username",
			Placeholder: "Enter your username",
		})

		@label.Label(label.Props{For: "email", Error: "Email is required"}) {
			Email
		}
		@input.Input(input.Props{
			ID:       "email",
			Name:     "email",
			Type:     input.TypeEmail,
			HasError: true,
		})
	</form>
}
```
