# Rating

> Interactive rating component for capturing user feedback and displaying scores, with star, heart, and emoji styles.

## Install

```bash
templui add rating
```

Dependencies auto-installed: icon

Add to your base layout `<head>`: `@rating.Script()`

## Update

```bash
templui add rating
```

To update all: `templui --installed add`

## API

### Enums

```go
type Style string

const (
	StyleStar  Style = "star"
	StyleHeart Style = "heart"
	StyleEmoji Style = "emoji"
)
```

### Props

```go
type Props struct {
	ID          string
	Class       string
	Attributes  templ.Attributes
	Value       float64
	ReadOnly    bool
	Precision   float64
	Name        string
	Form        string
	OnlyInteger bool
}
```

```go
type GroupProps struct {
	ID         string
	Class      string
	Attributes templ.Attributes
}
```

```go
type ItemProps struct {
	ID         string
	Class      string
	Attributes templ.Attributes
	Value      int
	Style      Style
}
```

## Components

| Function | Props | Description |
| -------- | ----- | ----------- |
| `rating.Rating(props ...Props)` | `Props` | Main rating container. Renders a hidden input when `Name` is set for form submission. Accepts children (typically a `Group`). |
| `rating.Group(props ...GroupProps)` | `GroupProps` | Horizontal flex container for grouping rating items. |
| `rating.Item(props ...ItemProps)` | `ItemProps` | Individual rating item displaying a star, heart, or emoji icon based on `Style`. |

## Composition

```
Rating
└── Group
    └── Item (repeated, one per rating level)
```

## Dependencies

- `icon` — used internally for star, heart, and emoji icons (`icon.Star`, `icon.Heart`, `icon.Angry`, `icon.Frown`, `icon.Meh`, `icon.Smile`, `icon.Laugh`)

## Context

No context passing.

## Data Attributes

| Attribute | Element | Description |
| --------- | ------- | ----------- |
| `data-tui-rating-component` | Rating container | Identifies the root rating element for JS initialization |
| `data-tui-rating-initial-value` | Rating container | Stores the initial float64 value |
| `data-tui-rating-precision` | Rating container | Stores the precision setting |
| `data-tui-rating-readonly` | Rating container | `"true"` or `"false"` — controls interactivity |
| `data-tui-rating-name` | Rating container | Form field name (present only when `Name` is set) |
| `data-tui-rating-onlyinteger` | Rating container | `"true"` or `"false"` — restricts to integer values |
| `data-tui-rating-current` | Rating container | Set by JS to track the live value |
| `data-tui-rating-hidden-input` | Hidden `<input>` | Marks the hidden input for JS binding |
| `data-tui-rating-item` | Item wrapper | Identifies each clickable rating item |
| `data-tui-rating-value` | Item wrapper | Integer value of this item |
| `data-tui-rating-item-foreground` | Item foreground div | Overlay element whose width is controlled by JS to show fill percentage |

## JS Events

| Event | Target | Detail | Description |
| ----- | ------ | ------ | ----------- |
| `rating-change` | Rating container | `{ name, value, maxValue }` | Fires on value change via user click |

## Usage

```templ
import "github.com/templui/templui/internal/components/rating"

// Interactive 5-star rating with half-star precision
@rating.Rating(rating.Props{
    Value:     3.5,
    Precision: 0.5,
    Name:      "score",
}) {
    @rating.Group() {
        for i := 1; i <= 5; i++ {
            @rating.Item(rating.ItemProps{
                Value: i,
                Style: rating.StyleStar,
            })
        }
    }
}
```
