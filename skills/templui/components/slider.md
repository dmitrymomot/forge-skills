# Slider

> Slider input component for selecting numeric values within a range.

## Install

```bash
templui add slider
```

Dependencies auto-installed: none

Add to your base layout `<head>`: `@slider.Script()`

## Update

```bash
templui add slider
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

type InputProps struct {
	ID         string
	Class      string
	Attributes templ.Attributes
	Name       string
	Min        int
	Max        int
	Step       int
	Value      int
	Disabled   bool
}

type ValueProps struct {
	ID         string
	Class      string
	Attributes templ.Attributes
	For        string // Corresponds to the ID of the Slider Input
}
```

## Components

| Function | Props | Description |
| -------- | ----- | ----------- |
| `slider.Slider(props ...Props)` | `Props` | Wrapper container for the slider input and value display. Renders children. |
| `slider.Input(props ...InputProps)` | `InputProps` | Range input element. Auto-generates ID if not provided. |
| `slider.Value(props ...ValueProps)` | `ValueProps` | Displays the current value of a linked slider input. Requires `For` to match the input ID. |
| `slider.Script()` | — | Emits the `<script>` tag for slider JS. Add once in `<head>`. |

## Composition

```
slider.Slider
├── slider.Input
└── slider.Value
```

`Slider` is the outer wrapper that accepts children. Place `Input` and `Value` inside it.

## Dependencies

None — only uses `utils` package.

## Context

No context passing.

## Data Attributes

| Attribute | Element | Description |
| --------- | ------- | ----------- |
| `data-tui-slider-wrapper` | `Slider` wrapper `<div>` | Identifies the slider container. |
| `data-tui-slider-input` | `Input` `<input>` | Identifies the range input for JS binding. |
| `data-tui-slider-value` | `Value` `<span>` | Identifies the value display element. |
| `data-tui-slider-value-for` | `Value` `<span>` | Links the value display to a specific input by ID. |

## Usage

```templ
@slider.Slider() {
    <div class="flex items-center gap-4">
        @slider.Input(slider.InputProps{
            ID:    "volume",
            Name:  "volume",
            Min:   0,
            Max:   100,
            Step:  1,
            Value: 50,
        })
        @slider.Value(slider.ValueProps{
            For: "volume",
        })
    </div>
}
```
