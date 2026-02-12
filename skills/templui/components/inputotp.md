# Input OTP

> One-time password input component with auto-advance, paste support, and grouped slot layout.

## Install

```bash
templui add inputotp
```

Dependencies auto-installed: input

Add to your base layout `<head>`: `@inputotp.Script()`

## Update

```bash
templui add inputotp
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
	Value      string
	Name       string
	Form       string
	HasError   bool
}
```

| Prop | Type | Default | Description |
| ---- | ---- | ------- | ----------- |
| ID | `string` | `""` | HTML `id` for the hidden input. The container `<div>` receives `ID + "-container"`. |
| Class | `string` | `""` | Additional CSS classes merged via `utils.TwMerge` on the container. |
| Attributes | `templ.Attributes` | `nil` | Arbitrary HTML attributes spread onto the container `<div>`. |
| Value | `string` | `""` | Initial OTP value. Characters are distributed across slots on mount via JavaScript. |
| Name | `string` | `""` | HTML `name` attribute on the hidden `<input>` for form submission. |
| Form | `string` | `""` | Associates the component with a `<form>` by form ID. |
| HasError | `bool` | `false` | Sets `aria-invalid="true"` on the hidden input. |

```go
type GroupProps struct {
	ID         string
	Class      string
	Attributes templ.Attributes
}
```

| Prop | Type | Default | Description |
| ---- | ---- | ------- | ----------- |
| ID | `string` | `""` | HTML `id` attribute. |
| Class | `string` | `""` | Additional CSS classes merged via `utils.TwMerge`. |
| Attributes | `templ.Attributes` | `nil` | Arbitrary HTML attributes spread onto the group `<div>`. |

```go
type SlotProps struct {
	ID          string
	Class       string
	Attributes  templ.Attributes
	Index       int
	Type        string
	Placeholder string
	Disabled    bool
	HasError    bool
}
```

| Prop | Type | Default | Description |
| ---- | ---- | ------- | ----------- |
| ID | `string` | `""` | HTML `id` attribute on the wrapper `<div>`. |
| Class | `string` | `""` | Additional CSS classes merged via `utils.TwMerge` on the `<input>`. |
| Attributes | `templ.Attributes` | `nil` | Arbitrary HTML attributes spread onto the `<input>`. |
| Index | `int` | `0` | Zero-based position of this slot. Rendered as `data-tui-inputotp-index`. Must be sequential across all slots. |
| Type | `string` | `"text"` | HTML input type (e.g. `"text"`, `"password"`). Always uses `inputmode="numeric"`. |
| Placeholder | `string` | `""` | Placeholder character shown when the slot is empty. |
| Disabled | `bool` | `false` | Disables the slot input. |
| HasError | `bool` | `false` | Applies destructive/error styling and sets `aria-invalid="true"`. |

```go
type SeparatorProps struct {
	ID         string
	Class      string
	Attributes templ.Attributes
}
```

| Prop | Type | Default | Description |
| ---- | ---- | ------- | ----------- |
| ID | `string` | `""` | HTML `id` attribute. |
| Class | `string` | `""` | Additional CSS classes merged via `utils.TwMerge`. |
| Attributes | `templ.Attributes` | `nil` | Arbitrary HTML attributes spread onto the separator `<div>`. |

## Components

| Function | Props | Description |
| -------- | ----- | ----------- |
| `inputotp.InputOTP(props ...Props)` | `inputotp.Props` | Root container. Renders a hidden `<input>` that collects the combined slot values for form submission, plus a flex wrapper for children. |
| `inputotp.Group(props ...GroupProps)` | `inputotp.GroupProps` | Flex container that groups adjacent `Slot` components together. |
| `inputotp.Slot(props ...SlotProps)` | `inputotp.SlotProps` | Single-character `<input>` with `maxlength="1"`. Auto-advances focus on input and supports backspace navigation. |
| `inputotp.Separator(props ...SeparatorProps)` | `inputotp.SeparatorProps` | Visual separator (renders a dash `-`) placed between groups. |

## Composition

```
InputOTP
  <input type="hidden">          (collects combined value)
  Group
    Slot                          (single-character input)
    Slot
    Slot
  Separator                       (visual dash between groups)
  Group
    Slot
    Slot
    Slot
```

## Dependencies

- `input` -- listed as a registry dependency.
- `utils` -- `TwMerge`, `If`.

## Context

No context passing.

## Data Attributes

| Attribute | Set On | Value | Description |
| --------- | ------ | ----- | ----------- |
| `data-tui-inputotp` | Container `<div>` | (presence) | Marks the root OTP container. JavaScript scopes all slot logic within this element. |
| `data-tui-inputotp-value` | Container `<div>` | Initial OTP string | Provides the initial value. JavaScript distributes characters across slots on mount. |
| `data-tui-inputotp-value-target` | Hidden `<input>` | (presence) | Identifies the hidden input that stores the combined slot values for form submission. |
| `data-tui-inputotp-slot` | Slot `<input>` | (presence) | Marks an element as an OTP slot input. JavaScript attaches input, keydown, focus, and paste handlers. |
| `data-tui-inputotp-index` | Slot `<input>` | Slot index (e.g. `"0"`, `"1"`) | Zero-based position used to order slots and distribute initial/pasted values. |

## Usage

```templ
package pages

import "github.com/templui/templui/internal/components/inputotp"

templ VerificationForm() {
	<form method="post" action="/verify">
		@inputotp.InputOTP(inputotp.Props{
			ID:   "otp-code",
			Name: "code",
		}) {
			@inputotp.Group() {
				@inputotp.Slot(inputotp.SlotProps{Index: 0})
				@inputotp.Slot(inputotp.SlotProps{Index: 1})
				@inputotp.Slot(inputotp.SlotProps{Index: 2})
			}
			@inputotp.Separator()
			@inputotp.Group() {
				@inputotp.Slot(inputotp.SlotProps{Index: 3})
				@inputotp.Slot(inputotp.SlotProps{Index: 4})
				@inputotp.Slot(inputotp.SlotProps{Index: 5})
			}
		}
		<button type="submit">Verify</button>
	</form>
}
```
