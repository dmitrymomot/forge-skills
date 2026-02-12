# Input

> Text input field with support for multiple HTML input types, error states, and an automatic password visibility toggle.

## Install

```bash
templui add input
```

Dependencies auto-installed: button, icon

Add to your base layout `<head>`: `@input.Script()`

## Update

```bash
templui add input
```

To update all: `templui --installed add`

## API

### Enums

```go
type Type string

const (
	TypeText     Type = "text"
	TypePassword Type = "password"
	TypeEmail    Type = "email"
	TypeNumber   Type = "number"
	TypeTel      Type = "tel"
	TypeURL      Type = "url"
	TypeSearch   Type = "search"
	TypeDate     Type = "date"
	TypeDateTime Type = "datetime-local"
	TypeTime     Type = "time"
	TypeFile     Type = "file"
	TypeColor    Type = "color"
	TypeWeek     Type = "week"
	TypeMonth    Type = "month"
)
```

### Props

```go
type Props struct {
	ID               string
	Class            string
	Attributes       templ.Attributes
	Name             string
	Type             Type
	Form             string
	Placeholder      string
	Value            string
	Disabled         bool
	Readonly         bool
	FileAccept       string
	HasError         bool
	NoTogglePassword bool
}
```

| Prop | Type | Default | Description |
| ---- | ---- | ------- | ----------- |
| ID | `string` | `utils.RandomID()` | HTML `id` attribute. Auto-generated if empty. |
| Class | `string` | `""` | Additional CSS classes merged via `utils.TwMerge`. |
| Attributes | `templ.Attributes` | `nil` | Arbitrary HTML attributes spread onto the `<input>`. |
| Name | `string` | `""` | HTML `name` attribute. |
| Type | `Type` | `TypeText` | Input type. Controls HTML `type` attribute. |
| Form | `string` | `""` | Associates the input with a `<form>` by form ID. |
| Placeholder | `string` | `""` | Placeholder text. |
| Value | `string` | `""` | Initial value. |
| Disabled | `bool` | `false` | Disables the input. |
| Readonly | `bool` | `false` | Makes the input read-only. |
| FileAccept | `string` | `""` | Accepted file types (only applied when `Type` is `TypeFile`). |
| HasError | `bool` | `false` | Applies destructive/error styling and sets `aria-invalid="true"`. |
| NoTogglePassword | `bool` | `false` | When `true`, hides the password visibility toggle button for password inputs. |

## Components

| Function | Props | Description |
| -------- | ----- | ----------- |
| `input.Input(props ...Props)` | `input.Props` | Renders an `<input>` wrapped in a relative `<div>`. For password type, automatically includes a visibility toggle button. |

## Composition

```
Input
  <input>
  (if Type == TypePassword && !NoTogglePassword)
    button.Button  (toggle visibility)
      icon.Eye     (visible state)
      icon.EyeOff  (hidden state)
```

## Dependencies

- `button` -- used for the password visibility toggle button.
- `icon` -- provides `Eye` and `EyeOff` icons for the toggle.
- `utils` -- `TwMerge`, `RandomID`, `If`.

## Context

No context passing.

## Data Attributes

| Attribute | Set On | Value | Description |
| --------- | ------ | ----- | ----------- |
| `data-tui-input-toggle-password` | Toggle `button.Button` | The input's `ID` | Links the toggle button to its password input. JavaScript listens for clicks on elements with this attribute to swap the input type between `password` and `text`. |

## Usage

```templ
package pages

import "github.com/templui/templui/internal/components/input"

templ LoginForm() {
	<form method="post" action="/login">
		<label for="email">Email</label>
		@input.Input(input.Props{
			ID:          "email",
			Name:        "email",
			Type:        input.TypeEmail,
			Placeholder: "you@example.com",
		})

		<label for="password">Password</label>
		@input.Input(input.Props{
			ID:          "password",
			Name:        "password",
			Type:        input.TypePassword,
			Placeholder: "Enter your password",
		})

		<button type="submit">Sign In</button>
	</form>
}
```
