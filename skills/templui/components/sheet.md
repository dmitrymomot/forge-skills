# Sheet

> A panel that slides in from the edge of the screen, built on top of the Dialog component with side-specific positioning and slide animations.

## Install

```bash
templui add sheet
```

Dependencies auto-installed: dialog

## Update

```bash
templui add sheet
```

To update all: `templui --installed add`

## API

### Enums

```go
type Side string

const (
	SideTop    Side = "top"
	SideRight  Side = "right"
	SideBottom Side = "bottom"
	SideLeft   Side = "left"
)
```

### Props

```go
type Props struct {
	ID               string
	Class            string
	Attributes       templ.Attributes
	Side             Side
	Open             bool
	DisableClickAway bool
	DisableESC       bool
}

type TriggerProps struct {
	ID         string
	Class      string
	Attributes templ.Attributes
	For        string // Reference to a specific sheet ID (for external triggers)
}

type ContentProps struct {
	ID              string
	Class           string
	Attributes      templ.Attributes
	HideCloseButton bool
	Side            Side //
	Open            bool // Initial open state for standalone usage
}

type HeaderProps struct {
	ID         string
	Class      string
	Attributes templ.Attributes
}

type FooterProps struct {
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

type CloseProps struct {
	ID         string
	Class      string
	Attributes templ.Attributes
	For        string
}
```

## Components

| Function | Props | Description |
| -------- | ----- | ----------- |
| `sheet.Sheet(props ...Props)` | `Props` | Root wrapper. Sets the side via context and renders a `dialog.Dialog` with sheet-specific data attributes. Defaults `Side` to `SideRight`. |
| `sheet.Trigger(props ...TriggerProps)` | `TriggerProps` | Button/element that opens the sheet. Delegates to `dialog.Trigger`. Use `For` to target a specific sheet ID. |
| `sheet.Content(props ...ContentProps)` | `ContentProps` | The sliding panel itself. Reads `Side` from context (or explicit prop) and applies side-specific positioning, sizing, border, and slide animation classes. Delegates to `dialog.Content`. |
| `sheet.Header(props ...HeaderProps)` | `HeaderProps` | Container for title and description inside the content panel. Adds `p-4` padding and left-aligned text. |
| `sheet.Title(props ...TitleProps)` | `TitleProps` | Sheet title text. Renders via `dialog.Title` with `text-base` styling. |
| `sheet.Description(props ...DescriptionProps)` | `DescriptionProps` | Sheet description text. Delegates directly to `dialog.Description`. |
| `sheet.Footer(props ...FooterProps)` | `FooterProps` | Footer area pinned to the bottom of the content panel via `mt-auto`. Provides `p-4` padding and vertical `gap-2` layout. |
| `sheet.Close(props ...CloseProps)` | `CloseProps` | Button/element that closes the sheet. Delegates to `dialog.Close`. Use `For` to target a specific sheet ID. |

## Composition

```
sheet.Sheet
├── sheet.Trigger
└── sheet.Content
    ├── sheet.Header
    │   ├── sheet.Title
    │   └── sheet.Description
    ├── {body content}
    └── sheet.Footer
        └── sheet.Close
```

## Dependencies

- `dialog` — all sheet sub-components delegate to corresponding dialog sub-components.
- `utils` — `TwMerge` and `MergeAttributes` for class and attribute composition.

## Context

| Key | Type | Description |
| --- | ---- | ----------- |
| `sideKey` (`"sheetSide"`) | `Side` | Set by `Sheet`, read by `Content` to determine slide direction when `ContentProps.Side` is not explicitly provided. |

## Data Attributes

| Attribute | Element | Values | Description |
| --------- | ------- | ------ | ----------- |
| `data-tui-sheet` | `Sheet` (root dialog) | `"true"` | Marks the dialog as a sheet instance. |
| `data-tui-sheet-side` | `Sheet` (root dialog) | `"top"`, `"right"`, `"bottom"`, `"left"` | Indicates which edge the sheet slides from. |
| `data-tui-sheet-content` | `Content` (dialog content) | `"true"` | Marks the content panel as a sheet content element. |
| `data-tui-sheet-side` | `Content` (dialog content) | `"top"`, `"right"`, `"bottom"`, `"left"` | Side indicator on the content element for CSS targeting. |

Inherited from dialog: `data-tui-dialog-open`, `data-tui-dialog-hidden` (used by animation classes).

## Usage

```templ
@sheet.Sheet(sheet.Props{Side: sheet.SideLeft}) {
  @sheet.Trigger() {
    @button.Button() {
      Open Settings
    }
  }
  @sheet.Content() {
    @sheet.Header() {
      @sheet.Title() {
        Settings
      }
      @sheet.Description() {
        Adjust your preferences below.
      }
    }
    <div class="p-4">
      <p>Sheet body content goes here.</p>
    </div>
    @sheet.Footer() {
      @sheet.Close() {
        @button.Button(button.Props{Variant: button.VariantOutline}) {
          Cancel
        }
      }
      @button.Button() {
        Save Changes
      }
    }
  }
}
```
