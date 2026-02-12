# Collapsible

> An interactive component that expands and collapses a content panel with smooth height transitions.

## Install

```bash
templui add collapsible
```

Dependencies auto-installed: none

Add to your base layout `<head>`: `@collapsible.Script()`

## Update

```bash
templui add collapsible
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
	Open       bool
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
}
```

## Components

| Function | Props | Description |
| -------- | ----- | ----------- |
| `Collapsible(props ...Props)` | `Props` | Root container `<div>` that manages the open/closed state. Auto-generates an ID if not provided. Sets `data-tui-collapsible-state` to `"open"` or `"closed"` based on the `Open` prop. |
| `Trigger(props ...TriggerProps)` | `TriggerProps` | Clickable region that toggles the collapsible content. Handles both click and keyboard (Space/Enter) events. Renders a `<div>` wrapper. |
| `Content(props ...ContentProps)` | `ContentProps` | The expandable/collapsible content panel. Uses a CSS grid `grid-rows-[0fr]`/`grid-rows-[1fr]` transition for smooth height animation. Content is wrapped in an inner `overflow-hidden` div. |

## Composition

```
Collapsible
├── Trigger   (toggle button area)
├── {always-visible content}
└── Content   (collapsible panel)
```

Place the `Trigger` and any always-visible content as direct children of `Collapsible`. The `Content` component wraps the elements that should expand/collapse. The `Trigger` does not need to be placed before `Content` -- it can appear anywhere inside `Collapsible`.

## Dependencies

None -- only uses `utils` package.

## Context

No context passing.

## Data Attributes

| Attribute | Element | Values | Description |
| --------- | ------- | ------ | ----------- |
| `data-tui-collapsible` | `Collapsible` | `"root"` | Marks the root collapsible container. |
| `data-tui-collapsible` | `Trigger` | `"trigger"` | Marks the trigger element. JavaScript delegates click/keyboard events to this. |
| `data-tui-collapsible` | `Content` | `"content"` | Marks the content panel. JavaScript toggles the `tui-collapsible-open` class on this element. |
| `data-tui-collapsible-state` | `Collapsible` | `"open"` \| `"closed"` | Tracks the current open/closed state. Toggled by the JavaScript on trigger activation. |

## Usage

```templ
@collapsible.Collapsible(collapsible.Props{
	Class: "w-[350px] space-y-2",
}) {
	<div class="flex items-center justify-between space-x-4 px-4">
		<h4 class="text-sm font-semibold">
			@axadrn starred 3 repositories
		</h4>
		@collapsible.Trigger() {
			@button.Button(button.Props{
				Variant: button.VariantGhost,
				Size:    button.SizeIcon,
				Class:   "size-8",
			}) {
				@icon.ChevronsUpDown(icon.Props{Class: "size-4"})
				<span class="sr-only">Toggle</span>
			}
		}
	</div>
	<div class="rounded-md border px-4 py-2 font-mono text-sm">
		github.com/a-h/templ
	</div>
	@collapsible.Content(collapsible.ContentProps{
		Class: "space-y-2",
	}) {
		<div class="rounded-md border px-4 py-2 font-mono text-sm">
			github.com/charmbracelet/bubbletea
		</div>
		<div class="rounded-md border px-4 py-2 font-mono text-sm">
			github.com/labstack/echo
		</div>
	}
}
```
