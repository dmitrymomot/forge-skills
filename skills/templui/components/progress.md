# Progress

> A horizontal progress bar with configurable size, variant, label, and percentage display.

## Install

```bash
templui add progress
```

Dependencies auto-installed: none

Add to your base layout `<head>`:
```html
<script defer src={ "/components/js/progress/progress.min.js?v=" + utils.ScriptVersion }></script>
```

## Update

```bash
templui add progress
```

To update all: `templui --installed add`

## API

### Enums

```go
type Size string

const (
    SizeSm Size = "sm"
    SizeLg Size = "lg"
)
```

```go
type Variant string

const (
    VariantDefault Variant = "default"
    VariantSuccess Variant = "success"
    VariantDanger  Variant = "danger"
    VariantWarning Variant = "warning"
)
```

### Props

```go
type Props struct {
    ID         string
    Class      string
    Attributes templ.Attributes
    Max        int
    Value      int
    Label      string
    ShowValue  bool
    Size       Size
    Variant    Variant
    BarClass   string
}
```

| Field | Type | Default | Description |
| ----- | ---- | ------- | ----------- |
| ID | string | auto-generated | Unique ID for the progress bar element |
| Class | string | "" | Additional CSS classes for the outer container |
| Attributes | templ.Attributes | nil | Additional HTML attributes spread on the outer `div` |
| Max | int | 100 | Maximum value of the progress bar (defaults to 100 if <= 0) |
| Value | int | 0 | Current progress value (clamped between 0 and Max) |
| Label | string | "" | Text label displayed above the bar on the left |
| ShowValue | bool | false | Whether to display the percentage value above the bar on the right |
| Size | Size | default (medium) | Height of the progress bar: SizeSm, SizeLg, or default |
| Variant | Variant | VariantDefault | Color variant: VariantDefault, VariantSuccess, VariantDanger, VariantWarning |
| BarClass | string | "" | Additional CSS classes for the inner progress indicator bar |

## Components

| Function | Props | Description |
| -------- | ----- | ----------- |
| `Progress(props ...Props)` | `Props` | Renders a progress bar with optional label and percentage display |

## Composition

Flat component -- no sub-components.

## Dependencies

None -- only uses `utils` package.

## Context

No context passing.

## Data Attributes

| Attribute | Element | Description |
| --------- | ------- | ----------- |
| `data-tui-progress-indicator` | Inner bar div | Identifies the bar element for JavaScript width animation |
| `data-tui-progress-observed` | Outer progress div | Added by JS to track which bars have MutationObserver attached |

The JavaScript reads `aria-valuenow` and `aria-valuemax` from the outer `div[role="progressbar"]` and sets the `width` style on the `[data-tui-progress-indicator]` element. A MutationObserver watches for attribute changes to update the bar width dynamically (useful with HTMX polling).

## Usage

```templ
import "github.com/templui/templui/components/progress"

// Basic progress bar with label and percentage
@progress.Progress(progress.Props{
    Value:     65,
    Label:     "Upload Progress",
    ShowValue: true,
    Variant:   progress.VariantSuccess,
    Size:      progress.SizeLg,
})
```
