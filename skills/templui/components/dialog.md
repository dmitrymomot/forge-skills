# Dialog

> A modal overlay component with backdrop, animated open/close transitions, click-away dismissal, ESC key support, and auto-focus management.

## Install

```bash
templui add dialog
```

Dependencies auto-installed: icon

Add to your base layout `<head>`: `@dialog.Script()`

## Update

```bash
templui add dialog
```

To update all: `templui --installed add`

## API

### Enums

No enums.

### Props

```go
type Props struct {
	ID               string
	Class            string
	Attributes       templ.Attributes
	DisableClickAway bool
	DisableESC       bool
	Open             bool
}

type TriggerProps struct {
	ID         string
	Class      string
	Attributes templ.Attributes
	For        string // Reference to a specific dialog ID (for external triggers)
}

type ContentProps struct {
	ID               string
	Class            string
	Attributes       templ.Attributes
	HideCloseButton  bool
	Open             bool // Initial open state for standalone usage (when no context)
	DisableAutoFocus bool
}

type CloseProps struct {
	ID         string
	Class      string
	Attributes templ.Attributes
	For        string // ID of the dialog to close (optional, defaults to closest dialog)
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
```

## Components

| Function | Props | Description |
| -------- | ----- | ----------- |
| `Dialog(props ...Props)` | `Props` | Root wrapper that establishes the dialog instance context. Generates a random instance ID if `ID` is not provided. Passes instance ID and open state to children via context. |
| `Trigger(props ...TriggerProps)` | `TriggerProps` | Clickable element that toggles the dialog open/closed. Uses `For` to target a specific dialog ID, otherwise inherits from parent `Dialog` context. Renders as a `<span>` with `contents` display. |
| `Content(props ...ContentProps)` | `ContentProps` | The modal panel and its backdrop overlay. Renders a fixed-position backdrop and a centered content container with scale/opacity transition animations. Includes a built-in close button (X icon) unless `HideCloseButton` is set. |
| `Close(props ...CloseProps)` | `CloseProps` | Clickable element that closes the dialog. Uses `For` to target a specific dialog ID, otherwise closes the closest parent dialog. Renders as a `<span>` with `contents` display. |
| `Header(props ...HeaderProps)` | `HeaderProps` | Semantic header section inside the content area. Renders a flex column layout with centered text on small screens and left-aligned on larger screens. |
| `Footer(props ...FooterProps)` | `FooterProps` | Semantic footer section inside the content area. Renders a reversed column layout on small screens and a right-justified row on larger screens. |
| `Title(props ...TitleProps)` | `TitleProps` | Dialog title rendered as an `<h2>` with semibold large text styling. |
| `Description(props ...DescriptionProps)` | `DescriptionProps` | Dialog description rendered as a `<p>` with muted small text styling. |

## Composition

```
Dialog
├── Trigger       (opens/closes the dialog)
└── Content       (backdrop + centered modal panel)
    ├── Header
    │   ├── Title
    │   └── Description
    ├── {body content}
    └── Footer
        └── Close  (wraps action buttons to close the dialog)
```

`Trigger` and `Close` can also be used outside the `Dialog` wrapper by setting their `For` prop to the target dialog's `ID`.

## Dependencies

- `icon` — used for the X close button icon inside `Content`.
- `utils` — used for `TwMerge` class merging and `RandomID` generation.

## Context

| Key | Type | Set By | Used By | Description |
| --- | ---- | ------ | ------- | ----------- |
| `dialogInstance` | `string` | `Dialog` | `Trigger`, `Content` | The unique instance ID for the dialog, used to associate triggers, backdrop, and content. |
| `dialogOpen` | `bool` | `Dialog` | `Content` | The initial open state, passed from `Props.Open` to `Content` for rendering the correct initial visibility. |

## Data Attributes

| Attribute | Element | Description |
| --------- | ------- | ----------- |
| `data-tui-dialog` | `Dialog` wrapper | Marks the root dialog container. |
| `data-dialog-instance` | All dialog elements | Instance ID linking trigger, backdrop, content, and close elements together. |
| `data-tui-dialog-disable-click-away` | `Dialog` wrapper | When present, clicking the backdrop does not close the dialog. |
| `data-tui-dialog-disable-esc` | `Dialog` wrapper | When present, pressing ESC does not close the dialog. |
| `data-tui-dialog-trigger` | `Trigger` | Marks the trigger element. Value is the target dialog instance ID. |
| `data-tui-dialog-trigger-open` | `Trigger` | Tracks whether the associated dialog is currently open (`"true"` or `"false"`). |
| `data-tui-dialog-backdrop` | `Content` backdrop | Marks the backdrop overlay element. |
| `data-tui-dialog-content` | `Content` panel | Marks the modal content panel. |
| `data-tui-dialog-open` | Backdrop and content | Controls visibility and animation state (`"true"` or `"false"`). |
| `data-tui-dialog-hidden` | Backdrop and content | When `"true"`, applies `!hidden` to fully remove from layout when closed. |
| `data-tui-dialog-disable-autofocus` | `Content` panel | When present, prevents auto-focusing the first focusable element on open. |
| `data-tui-dialog-close` | Close button / `Close` | Marks a close trigger. Value is the target dialog instance ID (optional). |

## JavaScript API

The dialog script exposes a global API at `window.tui.dialog`:

| Method | Signature | Description |
| ------ | --------- | ----------- |
| `open` | `open(dialogId: string)` | Programmatically open a dialog by instance ID. |
| `close` | `close(dialogId: string)` | Programmatically close a dialog by instance ID. |
| `toggle` | `toggle(dialogId: string)` | Toggle a dialog open or closed. |
| `isOpen` | `isOpen(dialogId: string): boolean` | Check whether a dialog is currently open. |

## Usage

```templ
// Confirmation dialog with header, description, and footer actions
@dialog.Dialog(dialog.Props{ID: "confirm-delete"}) {
	@dialog.Trigger() {
		@button.Button(button.Props{Variant: button.VariantDestructive}) {
			Delete Item
		}
	}
	@dialog.Content() {
		@dialog.Header() {
			@dialog.Title() {
				Are you sure?
			}
			@dialog.Description() {
				This action cannot be undone. This will permanently delete
				the item from your account.
			}
		}
		@dialog.Footer() {
			@dialog.Close() {
				@button.Button(button.Props{Variant: button.VariantOutline}) {
					Cancel
				}
			}
			@button.Button(button.Props{Variant: button.VariantDestructive}) {
				Confirm Delete
			}
		}
	}
}
```
