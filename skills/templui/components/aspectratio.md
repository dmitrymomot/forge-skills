# AspectRatio

> Constrains child content to a specified aspect ratio using a relative/absolute positioning wrapper.

## Install

```bash
templui add aspectratio
```

Dependencies auto-installed: none

## Update

```bash
templui add aspectratio
```

To update all: `templui --installed add`

## API

### Enums

```go
type Ratio string

const (
	RatioAuto     Ratio = "auto"
	RatioSquare   Ratio = "square"
	RatioVideo    Ratio = "video"
	RatioPortrait Ratio = "portrait"
	RatioWide     Ratio = "wide"
)
```

### Props

```go
type Props struct {
	ID         string
	Class      string
	Attributes templ.Attributes
	Ratio      Ratio
}
```

## Components

| Function | Props | Description |
| -------- | ----- | ----------- |
| `AspectRatio(props ...Props)` | `Props` | Renders a div that constrains its children to the given aspect ratio. Accepts children via `{ children... }`. |

## Composition

Flat component — no sub-components. The component wraps children inside an absolutely-positioned inner `div`.

## Dependencies

None — only uses `utils` package.

## Context

No context passing.

## Data Attributes

No data attributes.

## Usage

```templ
import "github.com/templui/templui/internal/components/aspectratio"

@aspectratio.AspectRatio(aspectratio.Props{
    Ratio: aspectratio.RatioVideo,
    Class: "overflow-hidden rounded-lg",
}) {
    <img src="/hero.jpg" alt="Hero" class="h-full w-full object-cover"/>
}
```
