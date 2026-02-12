# SelectBox

> Searchable select dropdown with single/multiple selection, pill tags, keyboard navigation, and form integration.

## Install

```bash
templui add selectbox
```

Dependencies auto-installed: button, icon, popover, input

Add to your base layout `<head>`:
```
@selectbox.Script()
@popover.Script()
```

## Update

```bash
templui add selectbox
```

To update all installed components:

```bash
templui --installed add
```

## API

### Enums

No enums.

### Props

```go
type Props struct {
	ID         string
	Class      string
	Attributes templ.Attributes
	Multiple   bool
}

type TriggerProps struct {
	ID                string
	Class             string
	Attributes        templ.Attributes
	Name              string
	Form              string
	Disabled          bool
	HasError          bool
	Multiple          bool
	ShowPills         bool
	SelectedCountText string
}

type ValueProps struct {
	ID          string
	Class       string
	Attributes  templ.Attributes
	Placeholder string
	Multiple    bool
}

type ContentProps struct {
	ID                string
	Class             string
	Attributes        templ.Attributes
	NoSearch          bool
	SearchPlaceholder string
}

type GroupProps struct {
	ID         string
	Class      string
	Attributes templ.Attributes
}

type LabelProps struct {
	ID         string
	Class      string
	Attributes templ.Attributes
}

type ItemProps struct {
	ID         string
	Class      string
	Attributes templ.Attributes
	Value      string
	Selected   bool
	Disabled   bool
}
```

## Components

| Function | Props | Description |
| -------- | ----- | ----------- |
| `SelectBox(props ...Props)` | `Props` | Root container for the select box. Sets up context for content ID sharing. |
| `Trigger(props ...TriggerProps)` | `TriggerProps` | Button that opens the dropdown. Contains hidden input for form submission, clear button, and chevron icon. |
| `Value(props ...ValueProps)` | `ValueProps` | Display area for the currently selected value(s) or placeholder text. |
| `Content(props ...ContentProps)` | `ContentProps` | Dropdown popover that holds the search input and selectable options. |
| `Group(props ...GroupProps)` | `GroupProps` | Container for grouping related options together. |
| `Label(props ...LabelProps)` | `LabelProps` | Non-selectable label text within a group. |
| `Item(props ...ItemProps)` | `ItemProps` | Individual selectable option within the dropdown. |

## Composition

```
SelectBox
├── Trigger
│   └── Value
└── Content
    └── Group (optional, repeatable)
        ├── Label (optional)
        └── Item (repeatable)
```

## Dependencies

- `button` — Trigger renders as a styled button
- `icon` — ChevronDown, CircleX (clear), Check (selected indicator), Search (search input icon)
- `popover` — Content and Trigger wrap popover components for dropdown behavior
- `input` — Search input inside the Content dropdown

## Context

| Key | Type | Description |
| --- | ---- | ----------- |
| `contentIDKey` | `contextKey("contentID")` | Shares the generated content popover ID from `SelectBox` to `Trigger` and `Content`. |

## Data Attributes

| Attribute | Element | Description |
| --------- | ------- | ----------- |
| `data-tui-selectbox-content-id` | Trigger button | Links the trigger to its content popover by ID. |
| `data-tui-selectbox-multiple` | Trigger button | `"true"` when multiple selection is enabled. |
| `data-tui-selectbox-show-pills` | Trigger button | `"true"` when selected items display as pill tags. |
| `data-tui-selectbox-selected-count-text` | Trigger button | Template string for count display, e.g. `"{n} items selected"`. |
| `data-tui-selectbox-hidden-input` | Hidden input | Marks the hidden input for reactive value binding. |
| `data-tui-selectbox-clear-trigger` | Clear icon span | Marks the clear-selection button. |
| `data-tui-selectbox-chevron` | Chevron icon span | Marks the dropdown chevron indicator. |
| `data-tui-selectbox-placeholder` | Value span | Stores the placeholder text for reset. |
| `data-tui-selectbox-search` | Search input | Marks the search input for filtering items. |
| `data-tui-selectbox-value` | Item div | The option value submitted with forms. |
| `data-tui-selectbox-selected` | Item div | `"true"` or `"false"` reflecting selection state. |
| `data-tui-selectbox-disabled` | Item div | `"true"` when the item cannot be selected. |
| `data-tui-selectbox-pill-remove` | Pill remove button | Marks the remove button inside a pill tag (JS-generated). |

## Usage

### Single selection

```templ
@selectbox.SelectBox() {
    @selectbox.Trigger(selectbox.TriggerProps{
        Name: "country",
    }) {
        @selectbox.Value(selectbox.ValueProps{
            Placeholder: "Select a country",
        })
    }
    @selectbox.Content() {
        @selectbox.Group() {
            @selectbox.Label() {
                Countries
            }
            @selectbox.Item(selectbox.ItemProps{Value: "us"}) {
                United States
            }
            @selectbox.Item(selectbox.ItemProps{Value: "gb"}) {
                United Kingdom
            }
            @selectbox.Item(selectbox.ItemProps{Value: "de"}) {
                Germany
            }
        }
    }
}
```

### Multiple selection with pills

```templ
@selectbox.SelectBox() {
    @selectbox.Trigger(selectbox.TriggerProps{
        Name:              "tags",
        ShowPills:         true,
        SelectedCountText: "{n} tags selected",
    }) {
        @selectbox.Value(selectbox.ValueProps{
            Placeholder: "Select tags",
        })
    }
    @selectbox.Content(selectbox.ContentProps{
        SearchPlaceholder: "Search tags...",
    }) {
        @selectbox.Item(selectbox.ItemProps{Value: "go", Selected: true}) {
            Go
        }
        @selectbox.Item(selectbox.ItemProps{Value: "templ"}) {
            Templ
        }
        @selectbox.Item(selectbox.ItemProps{Value: "htmx"}) {
            HTMX
        }
        @selectbox.Item(selectbox.ItemProps{Value: "deprecated", Disabled: true}) {
            Deprecated (unavailable)
        }
    }
}
```
