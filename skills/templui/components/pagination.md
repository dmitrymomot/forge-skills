# Pagination

> Navigation component for paging through multi-page content, with previous/next buttons, numbered page links, and ellipsis indicators.

## Install

```bash
templui add pagination
```

Dependencies auto-installed: button, icon

## Update

```bash
templui add pagination
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

type ContentProps struct {
	ID         string
	Class      string
	Attributes templ.Attributes
}

type ItemProps struct {
	ID         string
	Class      string
	Attributes templ.Attributes
}

type LinkProps struct {
	ID         string
	Class      string
	Attributes templ.Attributes
	Href       string
	IsActive   bool
	Disabled   bool
}

type PreviousProps struct {
	ID         string
	Class      string
	Attributes templ.Attributes
	Href       string
	Disabled   bool
	Label      string
}

type NextProps struct {
	ID         string
	Class      string
	Attributes templ.Attributes
	Href       string
	Disabled   bool
	Label      string
}
```

### Helper Function

```go
func CreatePagination(currentPage, totalPages, maxVisible int) struct {
	CurrentPage int
	TotalPages  int
	Pages       []int
	HasPrevious bool
	HasNext     bool
}
```

Returns a struct with computed pagination state. `maxVisible` controls how many page numbers are shown (defaults to 5 if less than 1). Clamps `currentPage` and `totalPages` to valid ranges.

## Components

| Function | Props | Description |
| -------- | ----- | ----------- |
| `pagination.Pagination(props ...Props)` | `Props` | Root `<nav>` wrapper with `role="navigation"` and `aria-label="pagination"`. Accepts children. |
| `pagination.Content(props ...ContentProps)` | `ContentProps` | `<ul>` container that lays out page items in a horizontal flex row with gap. Accepts children. |
| `pagination.Item(props ...ItemProps)` | `ItemProps` | `<li>` wrapper for individual pagination entries. Accepts children. |
| `pagination.Link(props ...LinkProps)` | `LinkProps` | Renders a page link as an icon-sized `button.Button`. Uses `VariantOutline` when `IsActive` is true, `VariantGhost` otherwise. Renders disabled button when `Disabled` is true. Accepts children (typically page number text). |
| `pagination.Previous(props ...PreviousProps)` | `PreviousProps` | Previous-page button with a `ChevronLeft` icon. Optional `Label` text displayed after the icon. Uses `button.VariantGhost`. |
| `pagination.Next(props ...NextProps)` | `NextProps` | Next-page button with a `ChevronRight` icon. Optional `Label` text displayed before the icon. Uses `button.VariantGhost`. |
| `pagination.Ellipsis()` | none | Renders an ellipsis icon (`icon.Ellipsis`) to indicate skipped page numbers. |

## Composition

```
Pagination
  Previous
  Content
    Item
      Link | Ellipsis
  Next
```

## Dependencies

- `button` -- `Previous`, `Next`, and `Link` render via `button.Button`
- `icon` -- `Previous` uses `icon.ChevronLeft`, `Next` uses `icon.ChevronRight`, `Ellipsis` uses `icon.Ellipsis`
- `utils` -- `utils.TwMerge` for class merging

## Context

No context passing.

## Data Attributes

No data attributes.

## Usage

```templ
import "github.com/templui/templui/internal/components/pagination"
import "fmt"

templ PaginatedList(currentPage, totalPages int) {
	{{ pg := pagination.CreatePagination(currentPage, totalPages, 5) }}

	@pagination.Pagination() {
		@pagination.Previous(pagination.PreviousProps{
			Href:     fmt.Sprintf("/items?page=%d", pg.CurrentPage-1),
			Disabled: !pg.HasPrevious,
			Label:    "Previous",
		})
		@pagination.Content() {
			if pg.Pages[0] > 1 {
				@pagination.Item() {
					@pagination.Link(pagination.LinkProps{Href: "/items?page=1"}) {
						1
					}
				}
				@pagination.Item() {
					@pagination.Ellipsis()
				}
			}
			for _, page := range pg.Pages {
				@pagination.Item() {
					@pagination.Link(pagination.LinkProps{
						Href:     fmt.Sprintf("/items?page=%d", page),
						IsActive: page == pg.CurrentPage,
					}) {
						{ fmt.Sprint(page) }
					}
				}
			}
			if pg.Pages[len(pg.Pages)-1] < pg.TotalPages {
				@pagination.Item() {
					@pagination.Ellipsis()
				}
				@pagination.Item() {
					@pagination.Link(pagination.LinkProps{
						Href: fmt.Sprintf("/items?page=%d", pg.TotalPages),
					}) {
						{ fmt.Sprint(pg.TotalPages) }
					}
				}
			}
		}
		@pagination.Next(pagination.NextProps{
			Href:     fmt.Sprintf("/items?page=%d", pg.CurrentPage+1),
			Disabled: !pg.HasNext,
			Label:    "Next",
		})
	}
}
```
