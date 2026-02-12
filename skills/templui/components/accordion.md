# Accordion

> A vertically stacked set of collapsible disclosure sections built on native HTML `<details>`/`<summary>` elements.

## Install

```bash
templui add accordion
```

Dependencies auto-installed: icon

## Update

```bash
templui add accordion
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

type ItemProps struct {
	ID         string
	Class      string
	Attributes templ.Attributes
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
| `Accordion(props ...Props)` | `Props` | Root wrapper `<div>` that contains accordion items. |
| `Item(props ...ItemProps)` | `ItemProps` | A single collapsible section. Renders a `<details>` element with `name="accordion"` so only one item is open at a time. |
| `Trigger(props ...TriggerProps)` | `TriggerProps` | The clickable header of an item. Renders a `<summary>` element with an auto-appended chevron icon that rotates on open. |
| `Content(props ...ContentProps)` | `ContentProps` | The expandable body of an item. Renders a `<div>` revealed when the parent `Item` is open. |

## Composition

```
Accordion
  Item
    Trigger
    Content
```

Each `Item` must contain exactly one `Trigger` and one `Content` as children. Multiple `Item` elements are placed inside a single `Accordion`.

## Dependencies

- `icon` -- uses `icon.ChevronDown` inside `Trigger` for the expand/collapse indicator.

## Context

No context passing.

## Data Attributes

No data attributes.

## Usage

```templ
@accordion.Accordion() {
  @accordion.Item() {
    @accordion.Trigger() {
      Is it accessible?
    }
    @accordion.Content() {
      Yes. It uses native HTML details/summary elements and supports keyboard navigation.
    }
  }
  @accordion.Item() {
    @accordion.Trigger() {
      Is it styled?
    }
    @accordion.Content() {
      Yes. It ships with default Tailwind CSS classes and can be customized via the Class prop.
    }
  }
}
```
