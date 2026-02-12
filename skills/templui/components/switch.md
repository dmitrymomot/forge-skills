# Switch

> A toggle switch control rendered as a styled checkbox input with a sliding thumb indicator.

## Install

```bash
templui add switch
```

Dependencies auto-installed: none

## Update

```bash
templui add switch
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
	Name       string
	Value      string
	Disabled   bool
	Checked    bool
	Form       string
}
```

**Note:** Because `switch` is a reserved Go keyword, the package must be imported with an alias:

```go
switchcomp "github.com/templui/templui/internal/components/switch"
```

## Components

| Function | Props | Description |
| -------- | ----- | ----------- |
| `Switch(props ...Props)` | `Props` | Renders a toggle switch with a hidden checkbox and a visual sliding thumb. Props are variadic; omitting them uses zero-value defaults. |

## Composition

Flat component — no sub-components.

The `Switch` renders a `<label>` wrapping a hidden `<input type="checkbox" role="switch">` and a visual `<div>` thumb element. Pair with a `label.Label` or `form.Label` component for accessible labeling.

## Dependencies

None — only uses `utils` package.

## Context

No context passing.

## Data Attributes

No data attributes.

## Usage

```templ
package pages

import (
	"github.com/templui/templui/internal/components/label"
	switchcomp "github.com/templui/templui/internal/components/switch"
)

templ SettingsPage() {
	<div class="flex items-center gap-2">
		@switchcomp.Switch(switchcomp.Props{
			ID:   "notifications",
			Name: "notifications",
		})
		@label.Label(label.Props{
			For: "notifications",
		}) {
			Enable Notifications
		}
	</div>
}
```
