# Dropdown

> A menu overlay triggered by click, built on the popover component, with support for items, groups, labels, separators, keyboard shortcuts, nested submenus, and auto-close behavior.

## Install

```bash
templui add dropdown
```

Dependencies auto-installed: popover

Add to your base layout `<head>`:
```
@dropdown.Script()
@popover.Script()
```

## Update

```bash
templui add dropdown
```

To update all: `templui --installed add`

## API

### Enums

```go
type Placement = popover.Placement

const (
	PlacementTop         = popover.PlacementTop         // "top"
	PlacementTopStart    = popover.PlacementTopStart     // "top-start"
	PlacementTopEnd      = popover.PlacementTopEnd       // "top-end"
	PlacementRight       = popover.PlacementRight        // "right"
	PlacementRightStart  = popover.PlacementRightStart   // "right-start"
	PlacementRightEnd    = popover.PlacementRightEnd     // "right-end"
	PlacementBottom      = popover.PlacementBottom       // "bottom"
	PlacementBottomStart = popover.PlacementBottomStart  // "bottom-start"
	PlacementBottomEnd   = popover.PlacementBottomEnd    // "bottom-end"
	PlacementLeft        = popover.PlacementLeft         // "left"
	PlacementLeftStart   = popover.PlacementLeftStart    // "left-start"
	PlacementLeftEnd     = popover.PlacementLeftEnd      // "left-end"
)
```

### Props

```go
type Props struct {
	ID string
}

type TriggerProps struct {
	ID         string
	Class      string
	Attributes templ.Attributes
}

type ContentProps struct {
	ID         string
	Class      string
	Attributes templ.Attributes
	Placement  Placement
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
	ID           string
	Class        string
	Attributes   templ.Attributes
	Disabled     bool
	Href         string
	Target       string
	PreventClose bool
}

type SeparatorProps struct {
	ID         string
	Class      string
	Attributes templ.Attributes
}

type ShortcutProps struct {
	ID         string
	Class      string
	Attributes templ.Attributes
}

type SubProps struct {
	ID         string
	Class      string
	Attributes templ.Attributes
}

type SubTriggerProps struct {
	ID         string
	Class      string
	Attributes templ.Attributes
}

type SubContentProps struct {
	ID         string
	Class      string
	Attributes templ.Attributes
}
```

## Components

| Function | Props | Description |
| -------- | ----- | ----------- |
| `Dropdown(props ...Props)` | `Props` | Root wrapper. Generates a content ID and passes it via context to link Trigger and Content. |
| `Trigger(props ...TriggerProps)` | `TriggerProps` | Click trigger that toggles the dropdown content. Wraps `popover.Trigger` with `TriggerTypeClick`. |
| `Content(props ...ContentProps)` | `ContentProps` | Floating panel that displays menu items. Wraps `popover.Content`. Defaults to `PlacementBottomStart`. |
| `Group(props ...GroupProps)` | `GroupProps` | Groups related items together with `role="group"`. |
| `Label(props ...LabelProps)` | `LabelProps` | Non-interactive label/heading within a group or menu. |
| `Item(props ...ItemProps)` | `ItemProps` | Individual menu item. Renders as `<a>` when `Href` is set, otherwise `<button>`. Supports disabled state and prevent-close behavior. |
| `Separator(props ...SeparatorProps)` | `SeparatorProps` | Horizontal rule between items with `role="separator"`. |
| `Shortcut(props ...ShortcutProps)` | `ShortcutProps` | Displays a keyboard shortcut hint inside an `Item`, aligned to the right. |
| `Sub(props ...SubProps)` | `SubProps` | Submenu wrapper. Generates a sub-content ID and passes it via context. |
| `SubTrigger(props ...SubTriggerProps)` | `SubTriggerProps` | Hover trigger for a submenu. Wraps `popover.Trigger` with `TriggerTypeHover`. Renders a chevron-right icon. |
| `SubContent(props ...SubContentProps)` | `SubContentProps` | Floating panel for submenu items. Wraps `popover.Content` with `PlacementRightStart`. |

## Composition

```
Dropdown
├── Trigger
├── Content
│   ├── Group
│   │   ├── Label
│   │   └── Item
│   │       └── Shortcut (optional, inside Item)
│   ├── Separator
│   ├── Item
│   │   └── Shortcut (optional)
│   └── Sub
│       ├── SubTrigger
│       └── SubContent
│           ├── Item
│           └── ...
```

## Dependencies

- `popover` — Trigger, Content, SubTrigger, and SubContent delegate to popover components.
- `utils` — TwMerge, RandomID, If, ScriptURL.

## Context

| Key | Type | Set by | Used by |
| --- | ---- | ------ | ------- |
| `contentIDKey` | `string` | `Dropdown` | `Trigger`, `Content` |
| `subContentIDKey` | `string` | `Sub` | `SubTrigger`, `SubContent` |

## Data Attributes

| Attribute | Element | Purpose |
| --------- | ------- | ------- |
| `data-tui-dropdown-item` | `Item` (button/anchor) | Identifies clickable menu items for auto-close JS behavior. |
| `data-tui-dropdown-prevent-close="true"` | `Item` | When set, clicking the item does not auto-close the dropdown. |
| `data-tui-dropdown-submenu` | `Sub` | Marks a submenu wrapper container. |
| `data-tui-dropdown-submenu-trigger` | `SubTrigger` (button) | Marks the submenu trigger button. |

## Usage

```templ
@dropdown.Dropdown() {
	@dropdown.Trigger() {
		@button.Button() {
			Options
		}
	}
	@dropdown.Content() {
		@dropdown.Label() {
			My Account
		}
		@dropdown.Separator()
		@dropdown.Group() {
			@dropdown.Item() {
				Profile
				@dropdown.Shortcut() { Ctrl+P }
			}
			@dropdown.Item() {
				Settings
				@dropdown.Shortcut() { Ctrl+S }
			}
		}
		@dropdown.Separator()
		@dropdown.Item(dropdown.ItemProps{Href: "/logout"}) {
			Log out
		}
		@dropdown.Sub() {
			@dropdown.SubTrigger() {
				More Options
			}
			@dropdown.SubContent() {
				@dropdown.Item() {
					Export
				}
				@dropdown.Item(dropdown.ItemProps{Disabled: true}) {
					Archive
				}
			}
		}
	}
}
```
