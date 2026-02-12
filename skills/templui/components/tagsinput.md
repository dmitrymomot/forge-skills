# TagsInput

> Interactive input field that allows users to enter, remove, and autocomplete tags with optional suggestions.

## Install

```bash
templui add tagsinput
```

Dependencies auto-installed: badge, input

Add to your base layout `<head>`:
```
@tagsinput.Script()
@popover.Script()
```

## Update

```bash
templui add tagsinput
```

To update all: `templui --installed add`

## API

### Enums

No enums.

### Props

```go
type Props struct {
	ID          string
	Name        string
	Value       []string
	Form        string
	Placeholder string
	Class       string
	HasError    bool
	Attributes  templ.Attributes
	Disabled    bool
	Readonly    bool
	Suggestions []string
}
```

## Components

| Function | Props | Description |
| -------- | ----- | ----------- |
| `TagsInput(props ...Props)` | `Props` | Tag input container with chips display, text input, hidden form fields, and optional suggestions popover. |

## Composition

```
TagsInput
├── <div data-tui-tagsinput-chips>         (flex-wrap row of tag chips, hidden when empty)
│   └── badge.Badge (per tag)              (chip with remove button)
│       ├── <span>                         (tag text)
│       └── <button data-tui-tagsinput-remove>  (remove icon)
├── <div>                                  (input row, acts as popover trigger when suggestions exist)
│   ├── input.Input                        (text input for typing new tags)
│   └── popover.Content (conditional)      (suggestion dropdown, only when Suggestions is non-empty)
│       └── <div data-tui-tagsinput-suggestion> (per suggestion item)
└── <div data-tui-tagsinput-hidden-inputs> (hidden container for form submission)
    └── <input type="hidden"> (per tag)    (name/value pairs for form data)
```

## Dependencies

- `badge` — renders each tag as a chip with remove button
- `input` — embedded text input for typing new tags
- `popover` — dropdown container for autocomplete suggestions (used internally when `Suggestions` is non-empty)
- `utils` — class merging and attribute utilities

## Context

No context passing.

## Data Attributes

| Attribute | Element | Purpose |
| --------- | ------- | ------- |
| `data-tui-tagsinput` | container div | Identifies the tags input root for JS initialization |
| `data-tui-tagsinput-name` | container div | Stores the `Name` prop for creating hidden inputs dynamically |
| `data-tui-tagsinput-form` | container div | Stores the `Form` prop for form association |
| `data-tui-tagsinput-suggestions-id` | container div | References the popover ID for the suggestions dropdown |
| `data-tui-tagsinput-chips` | chips wrapper div | Container for tag chip elements |
| `data-tui-tagsinput-chip` | badge element | Marks an individual tag chip |
| `data-tui-tagsinput-remove` | remove button | Marks the remove button inside a chip |
| `data-tui-tagsinput-text-input` | input element | Marks the text input for tag entry |
| `data-tui-tagsinput-suggestion` | suggestion div | Marks an individual suggestion item |
| `data-tui-tagsinput-suggestion-value` | suggestion div | Stores the suggestion's text value |
| `data-tui-tagsinput-hidden-inputs` | hidden container | Container for hidden `<input>` elements used in form submission |

## Usage

```templ
import "github.com/templui/templui/components/tagsinput"

// Basic tags input with pre-populated values
@tagsinput.TagsInput(tagsinput.Props{
	ID:          "networks",
	Name:        "networks",
	Placeholder: "Enter a network",
	Value:       []string{"127.0.0.1/32"},
})

// With autocomplete suggestions
@tagsinput.TagsInput(tagsinput.Props{
	ID:          "tags-with-suggestions",
	Name:        "os",
	Placeholder: "Add a tag...",
	Suggestions: []string{"Linux", "Windows", "macOS", "Ubuntu", "Debian"},
})
```
