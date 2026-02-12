# Alert

> Displays a callout message to attract user attention, with optional title, description, and icon.

## Install

```bash
templui add alert
```

Dependencies auto-installed: none

## Update

```bash
templui add alert
```

To update all: `templui --installed add`

## API

### Enums

```go
type Variant string

const (
	VariantDefault     Variant = "default"
	VariantDestructive Variant = "destructive"
)
```

### Props

```go
type Props struct {
	ID         string
	Class      string
	Attributes templ.Attributes
	Variant    Variant
}

type TitleProps struct {
	ID         string
	Class      string
	Attributes templ.Attributes
}

type DescriptionProps struct {
	ID         string
	Class      string
	Attributes templ.Attributes
}
```

## Components

| Function | Props | Description |
| -------- | ----- | ----------- |
| `Alert(props ...Props)` | `Props` | Root alert container. Renders a `<div>` with `role="alert"`. Accepts children (icons, Title, Description). |
| `Title(props ...TitleProps)` | `TitleProps` | Alert heading. Renders an `<h5>` inside the alert. |
| `Description(props ...DescriptionProps)` | `DescriptionProps` | Alert body text. Renders a `<div>` for descriptive content. |

## Composition

```
Alert
├── (optional icon — any SVG placed as direct child)
├── Title
└── Description
```

The grid layout auto-detects an SVG child via `has-[>svg]` and allocates a leading icon column.

## Dependencies

None — only uses `utils` package.

## Context

No context passing.

## Data Attributes

| Attribute | Element | Value |
| --------- | ------- | ----- |
| `data-slot="alert"` | Root `<div>` | Identifies the alert container |
| `data-slot="alert-title"` | `<h5>` | Identifies the title element |
| `data-slot="alert-description"` | `<div>` | Identifies the description element |

## Usage

```templ
import "github.com/templui/templui/components/alert"
import "github.com/templui/templui/components/icon"

@alert.Alert(alert.Props{Variant: alert.VariantDestructive}) {
	@icon.Icon(icon.Props{Name: "circle-alert"})
	@alert.Title() {
		Error
	}
	@alert.Description() {
		Your session has expired. Please log in again.
	}
}
```
