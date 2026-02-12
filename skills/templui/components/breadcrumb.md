# Breadcrumb

> Navigation breadcrumb trail showing the current page location within a hierarchy.

## Install

```bash
templui add breadcrumb
```

Dependencies auto-installed: icon

## Update

```bash
templui add breadcrumb
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

type ListProps struct {
	ID         string
	Class      string
	Attributes templ.Attributes
}

type ItemProps struct {
	ID         string
	Class      string
	Attributes templ.Attributes
	Current    bool
}

type LinkProps struct {
	ID         string
	Class      string
	Attributes templ.Attributes
	Href       string
}

type SeparatorProps struct {
	ID         string
	Class      string
	Attributes templ.Attributes
	UseCustom  bool
}
```

## Components

| Function | Props | Description |
| -------- | ----- | ----------- |
| `breadcrumb.Breadcrumb(props ...Props)` | `Props` | Root `<nav>` wrapper with `aria-label="Breadcrumb"`. |
| `breadcrumb.List(props ...ListProps)` | `ListProps` | Ordered list (`<ol>`) container for breadcrumb items. |
| `breadcrumb.Item(props ...ItemProps)` | `ItemProps` | List item (`<li>`) wrapping a link or page indicator. |
| `breadcrumb.Link(props ...LinkProps)` | `LinkProps` | Anchor (`<a>`) for a navigable breadcrumb step. |
| `breadcrumb.Separator(props ...SeparatorProps)` | `SeparatorProps` | Visual separator between items. Renders a `ChevronRight` icon by default; set `UseCustom: true` to supply children instead. |
| `breadcrumb.Page(props ...ItemProps)` | `ItemProps` | Terminal breadcrumb for the current page (`aria-current="page"`). |

## Composition

```
Breadcrumb
  List
    Item
      Link          // navigable ancestor
    Separator        // between items
    Item
      Link
    Separator
    Item
      Page           // current (non-link) leaf
```

## Dependencies

- `icon` -- used by `Separator` to render the default `ChevronRight` icon.

## Context

No context passing.

## Data Attributes

No data attributes.

## Usage

```templ
import "github.com/templui/templui/internal/components/breadcrumb"

@breadcrumb.Breadcrumb() {
  @breadcrumb.List() {
    @breadcrumb.Item() {
      @breadcrumb.Link(breadcrumb.LinkProps{Href: "/"}) {
        Home
      }
    }
    @breadcrumb.Separator()
    @breadcrumb.Item() {
      @breadcrumb.Link(breadcrumb.LinkProps{Href: "/products"}) {
        Products
      }
    }
    @breadcrumb.Separator()
    @breadcrumb.Item() {
      @breadcrumb.Page() {
        Widget
      }
    }
  }
}
```
