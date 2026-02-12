# Skeleton

> Animated placeholder element used to indicate loading content before data arrives.

## Install

```bash
templui add skeleton
```

Dependencies auto-installed: none

## Update

```bash
templui add skeleton
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
}
```

## Components

| Function | Props | Description |
| -------- | ----- | ----------- |
| `Skeleton(props ...Props)` | `Props` | Renders an animated pulse placeholder div |

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

import "github.com/templui/templui/internal/components/skeleton"

templ LoadingCard() {
	<div class="space-y-4 p-4">
		@skeleton.Skeleton(skeleton.Props{Class: "h-8 w-3/4"})
		@skeleton.Skeleton(skeleton.Props{Class: "h-4 w-full"})
		@skeleton.Skeleton(skeleton.Props{Class: "h-4 w-5/6"})
		@skeleton.Skeleton(skeleton.Props{Class: "h-32 w-full"})
	</div>
}
```
