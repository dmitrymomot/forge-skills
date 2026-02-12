# Form

> Structured form layout components for grouping labels, inputs, descriptions, and validation messages.

## Install

```bash
templui add form
```

Dependencies auto-installed: label

## Update

```bash
templui add form
```

To update all: `templui --installed add`

## API

### Enums

```go
type MessageVariant string

const (
	MessageVariantError MessageVariant = "error"
	MessageVariantInfo  MessageVariant = "info"
)
```

### Props

```go
type ItemProps struct {
	ID         string
	Class      string
	Attributes templ.Attributes
}

type LabelProps struct {
	ID            string
	Class         string
	Attributes    templ.Attributes
	For           string
	DisabledClass string
}

type DescriptionProps struct {
	ID         string
	Class      string
	Attributes templ.Attributes
}

type MessageProps struct {
	ID         string
	Class      string
	Attributes templ.Attributes
	Variant    MessageVariant
}
```

## Components

| Function | Props | Description |
| -------- | ----- | ----------- |
| `Item(props ...ItemProps)` | `ItemProps` | Vertical form item container with `space-y-2` spacing. Wraps a label, input, description, and message. |
| `ItemFlex(props ...ItemProps)` | `ItemProps` | Horizontal form item container with `flex space-x-2` layout. Used for inline elements like checkboxes. |
| `Label(props ...LabelProps)` | `LabelProps` | Form label that delegates to the `label.Label` component. |
| `Description(props ...DescriptionProps)` | `DescriptionProps` | Helper text rendered as a `<p>` in muted foreground color. |
| `Message(props ...MessageProps)` | `MessageProps` | Validation or info message rendered as a `<p>`. Variant controls color (error = red, info = blue). |

## Composition

```
Item / ItemFlex
  Label
  <input or other control>
  Description
  Message
```

`Item` provides vertical stacking (`space-y-2`). `ItemFlex` provides horizontal layout (`flex space-x-2`), typically used for checkbox/switch rows.

## Dependencies

- `label` -- `Label` delegates to `label.Label` internally.

## Context

No context passing.

## Data Attributes

No data attributes.

## Usage

```templ
@form.Item() {
  @form.Label(form.LabelProps{For: "email"}) {
    Email
  }
  @input.Input(input.Props{
    ID:          "email",
    Type:        input.TypeEmail,
    Placeholder: "you@example.com",
  })
  @form.Description() {
    We will never share your email.
  }
  @form.Message(form.MessageProps{Variant: form.MessageVariantError}) {
    Please enter a valid email address.
  }
}
```
