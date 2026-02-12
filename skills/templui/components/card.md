# Card

> A container component with header, title, description, content, and footer sections for grouping related information.

## Install

```bash
templui add card
```

Dependencies auto-installed: aspectratio

## Update

```bash
templui add card
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

type HeaderProps struct {
	ID         string
	Class      string
	Attributes templ.Attributes
}

type TitleProps struct {
	ID         string
	Class      string
	Attributes templ.Attributes
}

type DescriptionProps struct {
	ID         string
	Class      string
	Attributes templ.Attributes
}

type ContentProps struct {
	ID         string
	Class      string
	Attributes templ.Attributes
}

type FooterProps struct {
	ID         string
	Class      string
	Attributes templ.Attributes
}
```

## Components

| Function | Props | Description |
| -------- | ----- | ----------- |
| `Card(props ...Props)` | `Props` | Root card container. Renders a bordered, rounded `<div>` with shadow. |
| `Header(props ...HeaderProps)` | `HeaderProps` | Card header section with vertical flex layout and padding. |
| `Title(props ...TitleProps)` | `TitleProps` | Card title rendered as an `<h3>` with semibold styling. |
| `Description(props ...DescriptionProps)` | `DescriptionProps` | Card description rendered as a `<p>` with muted foreground color. |
| `Content(props ...ContentProps)` | `ContentProps` | Card body content area with padding. |
| `Footer(props ...FooterProps)` | `FooterProps` | Card footer section with horizontal flex layout. |

## Composition

```
Card
├── Header
│   ├── Title
│   └── Description
├── Content
└── Footer
```

## Dependencies

None — only uses `utils` package.

## Context

No context passing.

## Data Attributes

No data attributes.

## Usage

```templ
import "github.com/templui/templui/components/card"

@card.Card() {
	@card.Header() {
		@card.Title() {
			Notifications
		}
		@card.Description() {
			You have 3 unread messages.
		}
	}
	@card.Content() {
		<p>Your recent notifications will appear here.</p>
	}
	@card.Footer() {
		<button>Mark all as read</button>
	}
}
```
