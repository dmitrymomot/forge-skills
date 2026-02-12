# Icon

> Lucide icon rendering as inline SVG with caching, based on Lucide icons v0.544.0.

## Install

```bash
templui add icon
```

Dependencies auto-installed: none

## Update

```bash
templui add icon
```

To update all: `templui --installed add`

## API

### Constants

```go
const LucideVersion = "0.544.0"
```

### Props

```go
type Props struct {
	Size        int
	Color       string
	Fill        string
	Stroke      string
	StrokeWidth string // Stroke Width of Icon, Usage: "2.5"
	Class       string
}
```

**Defaults** (applied inside `generateSVG`):
- `Size`: `24`
- `Fill`: `"none"`
- `Stroke`: falls back to `Color`, then `"currentColor"`
- `StrokeWidth`: `"2"`

### Pre-defined Icon Variables

`icon_defs.go` exports 1636 pre-built icon variables. Each is declared as:

```go
var IconName = Icon("icon-slug")
```

First ~30 examples (alphabetical by slug):

```go
var AArrowDown = Icon("a-arrow-down")
var AArrowUp = Icon("a-arrow-up")
var ALargeSmall = Icon("a-large-small")
var Accessibility = Icon("accessibility")
var Activity = Icon("activity")
var AirVent = Icon("air-vent")
var Airplay = Icon("airplay")
var AlarmClock = Icon("alarm-clock")
var AlarmClockCheck = Icon("alarm-clock-check")
var AlarmClockMinus = Icon("alarm-clock-minus")
var AlarmClockOff = Icon("alarm-clock-off")
var AlarmClockPlus = Icon("alarm-clock-plus")
var AlarmSmoke = Icon("alarm-smoke")
var Album = Icon("album")
var AlignCenterHorizontal = Icon("align-center-horizontal")
var AlignCenterVertical = Icon("align-center-vertical")
var AlignEndHorizontal = Icon("align-end-horizontal")
var AlignEndVertical = Icon("align-end-vertical")
var AlignHorizontalDistributeCenter = Icon("align-horizontal-distribute-center")
var AlignHorizontalDistributeEnd = Icon("align-horizontal-distribute-end")
// ... and 1616 more
```

Common icons include: `Check`, `X`, `Plus`, `Minus`, `Search`, `Settings`, `User`, `Mail`, `Bell`, `Heart`, `Star`, `Home` (`House`), `ArrowLeft`, `ArrowRight`, `ArrowUp`, `ArrowDown`, `ChevronDown`, `ChevronUp`, `ChevronLeft`, `ChevronRight`, `Eye`, `EyeOff`, `Trash`, `Trash2`, `Edit` (use `Pencil`), `Copy`, `Download`, `Upload`, `ExternalLink`, `Menu`, `Filter` (use `Funnel`), `Info`, `CircleAlert`, `CircleCheck`, `CircleX`, `Loader`, `LoaderCircle`, `Github`, `Moon`, `Sun`.

## Components

| Function | Signature | Description |
| -------- | --------- | ----------- |
| `Icon` | `func Icon(name string) func(...Props) templ.Component` | Returns a factory function that produces a `templ.Component` rendering the named Lucide icon as inline SVG. Results are cached by name+props combination. |

The pre-defined variables (e.g. `icon.Check`, `icon.ArrowRight`) are the idiomatic way to use this component. Each variable already holds the `func(...Props) templ.Component` returned by `Icon()`.

## Composition

Flat component -- no sub-components.

## Dependencies

None -- only uses standard library (`context`, `fmt`, `io`, `sync`) and `github.com/a-h/templ`.

## Context

No context passing.

## Data Attributes

The rendered SVG includes `data-lucide="icon"` on every icon element.

## Usage

Render an icon with default settings:

```go
// In a templ template:
@icon.Check()
```

Render an icon with custom props:

```go
@icon.ArrowRight(icon.Props{
    Size:  16,
    Color: "red",
    Class: "inline-block",
})
```

Render by dynamic name (less common):

```go
@icon.Icon("chevron-down")(icon.Props{Size: 20})
```
