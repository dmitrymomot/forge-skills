# Textarea

> Multi-line text input with optional auto-resize behavior, error states, and configurable rows.

## Install

```bash
templui add textarea
```

Dependencies auto-installed: none

Add to your base layout `<head>`: `@textarea.Script()`

## Update

```bash
templui add textarea
```

To update all: `templui --installed add`

## API

### Enums

No enums.

### Props

```go
type Props struct {
	ID          string
	Class       string
	Attributes  templ.Attributes
	Name        string
	Value       string
	Form        string
	Placeholder string
	Rows        int
	AutoResize  bool
	Disabled    bool
	Readonly    bool
	HasError    bool
}
```

| Prop | Type | Default | Description |
| ---- | ---- | ------- | ----------- |
| ID | `string` | `utils.RandomID()` | HTML `id` attribute. Auto-generated if empty. |
| Class | `string` | `""` | Additional CSS classes merged via `utils.TwMerge`. |
| Attributes | `templ.Attributes` | `nil` | Arbitrary HTML attributes spread onto the `<textarea>`. |
| Name | `string` | `""` | HTML `name` attribute. |
| Value | `string` | `""` | Initial text content of the textarea. |
| Form | `string` | `""` | Associates the textarea with a `<form>` by form ID. |
| Placeholder | `string` | `""` | Placeholder text. |
| Rows | `int` | `0` | Number of visible text lines. When `0`, the attribute is omitted and the browser default applies. |
| AutoResize | `bool` | `false` | Enables automatic height resizing as the user types. Sets `data-tui-textarea-auto-resize="true"` and applies `overflow-hidden resize-none` styles. |
| Disabled | `bool` | `false` | Disables the textarea. |
| Readonly | `bool` | `false` | Makes the textarea read-only. |
| HasError | `bool` | `false` | Applies destructive/error styling and sets `aria-invalid="true"`. |

## Components

| Function | Props | Description |
| -------- | ----- | ----------- |
| `textarea.Textarea(props ...Props)` | `textarea.Props` | Renders a `<textarea>` element with styling, optional auto-resize, and error state support. |

## Composition

Flat component — no sub-components.

## Dependencies

None — only uses `utils` package.

## Context

No context passing.

## Data Attributes

| Attribute | Set On | Value | Description |
| --------- | ------ | ----- | ----------- |
| `data-tui-textarea` | `<textarea>` | _(presence)_ | Identifies the element as a templui textarea for JavaScript targeting. |
| `data-tui-textarea-auto-resize` | `<textarea>` | `"true"` | Only set when `AutoResize` is `true`. JavaScript listens for `input` events on textareas with this attribute and adjusts the element height to match its scroll height. |

## Usage

```templ
package pages

import "github.com/templui/templui/internal/components/textarea"

templ FeedbackForm() {
	<form method="post" action="/feedback">
		<label for="message">Your Message</label>
		@textarea.Textarea(textarea.Props{
			ID:          "message",
			Name:        "message",
			Placeholder: "Tell us what you think...",
			Rows:        4,
			AutoResize:  true,
		})

		<button type="submit">Send</button>
	</form>
}
```
