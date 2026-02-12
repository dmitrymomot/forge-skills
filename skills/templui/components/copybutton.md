# CopyButton

> Button component that copies content from a target element to the clipboard, with visual feedback and fallback support for older browsers.

## Install

```bash
templui add copybutton
```

Dependencies auto-installed: button, icon

Add to your base layout `<head>`: `@copybutton.Script()`

## Update

```bash
templui add copybutton
```

To update all: `templui --installed add`

## API

### Enums

No enums.

### Props

```go
type Props struct {
	ID       string           // Optional button ID
	Class    string           // Custom CSS classes
	Attrs    templ.Attributes // Additional HTML attributes
	TargetID string           // Required - ID of element to copy from
}
```

## Components

| Function | Props | Description |
| -------- | ----- | ----------- |
| `CopyButton(props Props)` | `Props` | Renders a ghost icon button that copies text content from the element identified by `TargetID` to the clipboard. Shows a clipboard icon by default and a check icon for 2 seconds after a successful copy. |

## Composition

Flat component — no sub-components.

Internally renders a `button.Button` (ghost variant, icon size) containing clipboard and check icons. The wrapping `<div>` carries `data-copy-button` and `data-target-id` attributes used by the JavaScript.

## Dependencies

- `button` — renders the underlying button element
- `icon` — provides `Clipboard` and `Check` icons
- `utils` — ID generation and class merging

## Context

No context passing.

## Data Attributes

| Attribute | Element | Description |
| --------- | ------- | ----------- |
| `data-copy-button` | wrapper `<div>` | Identifies the copy button for JS event delegation |
| `data-target-id` | wrapper `<div>` | Holds the ID of the element whose content will be copied |
| `data-copy-icon-clipboard` | `<span>` | Marks the clipboard icon (visible by default) |
| `data-copy-icon-check` | `<span>` | Marks the check icon (hidden, shown after copy) |

## Usage

```templ
package pages

import "your-project/components/copybutton"

templ CopyExample() {
	<div class="flex items-center gap-4">
		<div id="copy-target" class="text-sm text-muted-foreground">
			Hello, World!
		</div>
		@copybutton.CopyButton(copybutton.Props{
			TargetID: "copy-target",
		})
	</div>
}
```
