# Separator

> A visual divider that separates content sections, supporting horizontal and vertical orientations with optional label content and decoration styles.

## Install

```bash
templui add separator
```

Dependencies auto-installed: none

## Update

```bash
templui add separator
```

To update all: `templui --installed add`

## API

### Enums

```go
type Orientation string

const (
	OrientationHorizontal Orientation = "horizontal"
	OrientationVertical   Orientation = "vertical"
)

type Decoration string

const (
	DecorationDashed Decoration = "dashed"
	DecorationDotted Decoration = "dotted"
)
```

### Props

```go
type Props struct {
	ID          string
	Class       string
	Attributes  templ.Attributes
	Orientation Orientation
	Decoration  Decoration
}
```

## Components

| Function | Props | Description |
| -------- | ----- | ----------- |
| `Separator(props ...Props)` | `Props` | Renders a divider line with optional child content as a centered label. Defaults to horizontal orientation. |

## Composition

Flat component — no sub-components.

Children passed to `Separator` are rendered as a centered label on top of the divider line (e.g., text like "OR").

## Dependencies

None — only uses `utils` package.

## Context

No context passing.

## Data Attributes

No data attributes.

## Usage

```templ
// Horizontal separator with label
@separator.Separator() {
	OR
}

// Vertical dashed separator without label
@separator.Separator(separator.Props{
	Orientation: separator.OrientationVertical,
	Decoration:  separator.DecorationDashed,
})

// Horizontal dotted separator with custom class
@separator.Separator(separator.Props{
	Decoration: separator.DecorationDotted,
	Class:      "my-8",
}) {
	Section Break
}
```
