# Button

> Interactive button element that renders as `<button>` or `<a>` depending on whether an Href is provided.

## Install

```bash
templui add button
```

Dependencies auto-installed: none

## Update

```bash
templui add button
```

To update all installed components:

```bash
templui --installed add
```

## API

### Enums

```go
type Variant string

const (
	VariantDefault     Variant = "default"
	VariantDestructive Variant = "destructive"
	VariantOutline     Variant = "outline"
	VariantSecondary   Variant = "secondary"
	VariantGhost       Variant = "ghost"
	VariantLink        Variant = "link"
)
```

```go
type Type string

const (
	TypeButton Type = "button"
	TypeReset  Type = "reset"
	TypeSubmit Type = "submit"
)
```

```go
type Size string

const (
	SizeDefault Size = "default"
	SizeSm      Size = "sm"
	SizeLg      Size = "lg"
	SizeIcon    Size = "icon"
)
```

### Props

```go
type Props struct {
	ID         string
	Class      string
	Attributes templ.Attributes
	Variant    Variant
	Size       Size
	FullWidth  bool
	Href       string
	Target     string
	Disabled   bool
	Type       Type
	Form       string
}
```

## Components

| Function | Props   | Description                                                                      |
| -------- | ------- | -------------------------------------------------------------------------------- |
| `Button` | `Props` | Renders `<a>` when Href is set (and not Disabled), otherwise renders `<button>`. |

## Composition

Flat component -- no sub-components.

## Dependencies

None -- only uses `utils` package.

## Context

No context passing.

## Data Attributes

No data attributes.

## Usage

```go
// Standard button
@button.Button(button.Props{
    Variant: button.VariantDefault,
    Size:    button.SizeDefault,
}) {
    Click Me
}

// Destructive submit button tied to a form
@button.Button(button.Props{
    Type:    button.TypeSubmit,
    Variant: button.VariantDestructive,
    Form:    "delete-form",
}) {
    Delete
}

// Link-style button rendering as <a>
@button.Button(button.Props{
    Variant: button.VariantLink,
    Href:    "/settings",
}) {
    Go to Settings
}

// Full-width outline button
@button.Button(button.Props{
    Variant:   button.VariantOutline,
    FullWidth: true,
    Disabled:  true,
}) {
    Disabled Action
}
```
