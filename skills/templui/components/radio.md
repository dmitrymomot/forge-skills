# Radio

> A styled radio button input for single-selection within a group.

## Install

```bash
templui add radio
```

Dependencies auto-installed: none

## Update

```bash
templui add radio
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
	Form       string
	Disabled   bool
	Checked    bool
}
```

## Components

| Function | Props | Description |
| -------- | ----- | ----------- |
| `Radio(props ...Props)` | `Props` | Renders a styled `<input type="radio">` element. |

## Composition

Flat component — no sub-components.

## Dependencies

None — only uses `utils` package.

## Context

No context passing.

## Data Attributes

No data attributes.

## Usage

```templ
package pages

import "github.com/templui/templui/internal/components/radio"

templ ColorSelection() {
	<fieldset>
		<legend>Select a color</legend>
		<div class="flex flex-col gap-2">
			<label class="flex items-center gap-2">
				@radio.Radio(radio.Props{
					Name:    "color",
					Value:   "red",
					Checked: true,
				})
				Red
			</label>
			<label class="flex items-center gap-2">
				@radio.Radio(radio.Props{
					Name:  "color",
					Value: "green",
				})
				Green
			</label>
			<label class="flex items-center gap-2">
				@radio.Radio(radio.Props{
					Name:  "color",
					Value: "blue",
				})
				Blue
			</label>
		</div>
	</fieldset>
}
```
