# Badge

> Inline label for status, category, or count indicators.

## Install

```bash
templui add badge
```

Dependencies auto-installed: none

## Update

```bash
templui add badge
```

To update all: `templui --installed add`

## API

### Enums

```go
type Variant string

const (
	VariantDefault     Variant = "default"
	VariantSecondary   Variant = "secondary"
	VariantDestructive Variant = "destructive"
	VariantOutline     Variant = "outline"
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
```

## Components

| Function | Props | Description |
| -------- | ----- | ----------- |
| `Badge(props ...Props)` | `Props` | Renders an inline badge `<span>` with variant styling. Accepts children. |

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

import "github.com/templui/templui/internal/components/badge"

templ StatusBadges() {
	@badge.Badge(badge.Props{Variant: badge.VariantDefault}) {
		Active
	}
	@badge.Badge(badge.Props{Variant: badge.VariantSecondary}) {
		Pending
	}
	@badge.Badge(badge.Props{Variant: badge.VariantDestructive}) {
		Expired
	}
	@badge.Badge(badge.Props{Variant: badge.VariantOutline}) {
		Draft
	}
}
```
