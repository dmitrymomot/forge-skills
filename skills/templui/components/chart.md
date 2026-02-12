# Chart

> Chart component powered by Chart.js for rendering bar, line, pie, doughnut, and radar charts.

## Install

```bash
templui add chart
```

Dependencies auto-installed: none

Add to your base layout `<head>`: `@chart.Script()`

## Update

```bash
templui add chart
```

To update all: `templui --installed add`

## API

### Enums

```go
type Variant string

const (
	VariantBar      Variant = "bar"
	VariantLine     Variant = "line"
	VariantPie      Variant = "pie"
	VariantDoughnut Variant = "doughnut"
	VariantRadar    Variant = "radar"
)
```

### Props

```go
type Dataset struct {
	Label           string      `json:"label"`
	Data            []float64   `json:"data"`
	BorderWidth     int         `json:"borderWidth,omitempty"`
	BorderColor     interface{} `json:"borderColor,omitempty"`
	BackgroundColor interface{} `json:"backgroundColor,omitempty"`
	Tension         float64     `json:"tension,omitempty"`
	Fill            bool        `json:"fill,omitempty"`
	Stepped         bool        `json:"stepped,omitempty"`
}

type Options struct {
	Responsive bool `json:"responsive,omitempty"`
	Legend     bool `json:"legend,omitempty"`
}

type Data struct {
	Labels   []string  `json:"labels"`
	Datasets []Dataset `json:"datasets"`
}

type Config struct {
	Type        Variant  `json:"type"`
	Data        Data     `json:"data"`
	Options     Options  `json:"options,omitempty"`
	ShowLegend  bool     `json:"showLegend,omitempty"`
	ShowXAxis   bool     `json:"showXAxis"`
	ShowYAxis   bool     `json:"showYAxis"`
	ShowXLabels bool     `json:"showXLabels"`
	ShowYLabels bool     `json:"showYLabels"`
	ShowXGrid   bool     `json:"showXGrid"`
	ShowYGrid   bool     `json:"showYGrid"`
	Horizontal  bool     `json:"horizontal"`
	Stacked     bool     `json:"stacked"`
	YMin        *float64 `json:"yMin,omitempty"`
	YMax        *float64 `json:"yMax,omitempty"`
	BeginAtZero *bool    `json:"beginAtZero,omitempty"`
}

type Props struct {
	ID          string
	Variant     Variant
	Data        Data
	Options     Options
	ShowLegend  bool
	ShowXAxis   bool
	ShowYAxis   bool
	ShowXLabels bool
	ShowYLabels bool
	ShowXGrid   bool
	ShowYGrid   bool
	Horizontal  bool
	Stacked     bool
	YMin        *float64
	YMax        *float64
	BeginAtZero *bool
	Class       string
	Attributes  templ.Attributes
}
```

## Components

| Function | Props | Description |
| -------- | ----- | ----------- |
| `Chart(props ...Props)` | `Props` | Renders a chart canvas with the given configuration. Props are variadic; defaults apply when omitted. |

## Composition

Flat component — no sub-components.

Internally renders a `<div>` container wrapping a `<canvas>` element and a JSON `<script>` block holding the chart configuration.

## Dependencies

None — only uses `utils` package.

## Context

No context passing.

## Data Attributes

| Attribute | Element | Description |
| --------- | ------- | ----------- |
| `data-tui-chart-id` | `<canvas>` | Links the canvas to its JSON configuration `<script>` element. |

## Usage

```templ
package pages

import "yourapp/components/chart"

templ Dashboard() {
	@chart.Chart(chart.Props{
		Variant: chart.VariantBar,
		Data: chart.Data{
			Labels: []string{"Jan", "Feb", "Mar", "Apr", "May"},
			Datasets: []chart.Dataset{
				{
					Label:           "Revenue",
					Data:            []float64{120, 190, 300, 250, 420},
					BackgroundColor: "hsl(var(--primary))",
					BorderColor:     "hsl(var(--primary))",
					BorderWidth:     1,
				},
			},
		},
		ShowLegend:  true,
		ShowXAxis:   true,
		ShowYAxis:   true,
		ShowXLabels: true,
		ShowYLabels: true,
		ShowXGrid:   false,
		ShowYGrid:   true,
		Class:       "h-[300px]",
	})
}
```
