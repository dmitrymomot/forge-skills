# Tooltip

> A lightweight informational overlay that appears on hover, built on top of the popover component with tooltip-specific styling and positioning.

## Install

```bash
templui add tooltip
```

Dependencies auto-installed: popover

## Update

```bash
templui add tooltip
```

To update all: `templui --installed add`

## API

### Enums

```go
type Position string

const (
	PositionTop    Position = "top"
	PositionRight  Position = "right"
	PositionBottom Position = "bottom"
	PositionLeft   Position = "left"
)
```

### Props

```go
type Props struct {
	ID         string
	Class      string
	Attributes templ.Attributes
}

type TriggerProps struct {
	ID         string
	Class      string
	Attributes templ.Attributes
	For        string
}

type ContentProps struct {
	ID            string
	Class         string
	Attributes    templ.Attributes
	ShowArrow     bool
	Position      Position
	HoverDelay    int
	HoverOutDelay int
}
```

## Components

| Function | Props | Description |
| -------- | ----- | ----------- |
| `Tooltip(props ...Props)` | `Props` | Wrapper container that groups a trigger and content together. Renders children directly. |
| `Trigger(props ...TriggerProps)` | `TriggerProps` | The element that activates the tooltip on hover. Wraps children in a popover trigger with `TriggerTypeHover`. Use `For` to link to a specific `Content` by ID. |
| `Content(props ...ContentProps)` | `ContentProps` | The tooltip popup body. Renders inside a popover content with tooltip-specific styling (`bg-foreground text-background`). Defaults to top position. |

## Composition

```
Tooltip
├── Trigger  (hover-activated trigger element)
└── Content  (tooltip popup body)
```

`Trigger.For` must match `Content.ID` to associate them when they are not adjacent siblings within the same `Tooltip` wrapper.

## Dependencies

- `popover` — `Trigger` delegates to `popover.Trigger` (with `TriggerTypeHover`); `Content` delegates to `popover.Content`.
- `utils` — used for `TwMerge` in `Content` styling.

## Context

No context passing.

## Data Attributes

No tooltip-specific data attributes. The underlying popover component applies its own `data-tui-popover-*` attributes to the rendered elements.

## Usage

```templ
// Basic tooltip with arrow, positioned to the top (default)
@tooltip.Tooltip() {
	@tooltip.Trigger() {
		<button>Hover me</button>
	}
	@tooltip.Content(tooltip.ContentProps{
		ID:        "my-tooltip",
		ShowArrow: true,
	}) {
		This is a tooltip
	}
}

// Tooltip positioned to the right with custom delays
@tooltip.Tooltip() {
	@tooltip.Trigger(tooltip.TriggerProps{
		For: "info-tip",
	}) {
		<span class="underline cursor-help">What is this?</span>
	}
	@tooltip.Content(tooltip.ContentProps{
		ID:            "info-tip",
		Position:      tooltip.PositionRight,
		ShowArrow:     true,
		HoverDelay:    200,
		HoverOutDelay: 100,
	}) {
		Additional information about this feature.
	}
}
```
