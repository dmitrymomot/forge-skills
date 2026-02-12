# Checkbox

> Checkbox input with support for checked/disabled states, custom icons, indeterminate state, and parent-child group linking.

## Install

```bash
templui add checkbox
```

Dependencies auto-installed: icon

Add to your base layout `<head>`:

```go
@checkbox.Script()
```

## Update

```bash
templui add checkbox
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
	ID          string
	Class       string
	Attributes  templ.Attributes
	Name        string
	Value       string
	Disabled    bool
	Checked     bool
	Group       string
	GroupParent bool
	Form        string
	Icon        templ.Component
}
```

| Prop | Type | Default | Description |
| ---- | ---- | ------- | ----------- |
| ID | `string` | `""` | HTML `id` attribute for the `<input>`. |
| Class | `string` | `""` | Additional CSS classes merged via `utils.TwMerge`. |
| Attributes | `templ.Attributes` | `nil` | Arbitrary HTML attributes spread onto the `<input>`. |
| Name | `string` | `""` | HTML `name` attribute. |
| Value | `string` | `"on"` | HTML `value` attribute. Defaults to `"on"` when empty. |
| Disabled | `bool` | `false` | Disables the checkbox. |
| Checked | `bool` | `false` | Sets the checkbox as checked. |
| Group | `string` | `""` | Group name for parent-child checkbox linking. Sets `data-tui-checkbox-group`. |
| GroupParent | `bool` | `false` | Marks this checkbox as the parent of a group. When toggled, all children in the same group are checked/unchecked. Sets `data-tui-checkbox-parent`. |
| Form | `string` | `""` | Associates the checkbox with a `<form>` by form ID. |
| Icon | `templ.Component` | `nil` | Custom icon component shown when checked. Defaults to `icon.Check` (size 14) when nil. |

## Components

| Function | Props | Description |
| -------- | ----- | ----------- |
| `checkbox.Checkbox(props ...Props)` | `checkbox.Props` | Renders a styled checkbox `<input>` wrapped in a `<div>` with check and indeterminate icon overlays. |

## Composition

```
Checkbox
  <input type="checkbox">
  <div>  (checked overlay)
    Icon  (custom, or icon.Check default)
  <div>  (indeterminate overlay)
    icon.Minus
```

## Dependencies

- `icon` -- provides `Check` (default checked icon) and `Minus` (indeterminate icon).
- `utils` -- `TwMerge`.

## Context

No context passing.

## Data Attributes

| Attribute | Set On | Value | Description |
| --------- | ------ | ----- | ----------- |
| `data-tui-checkbox-group` | `<input>` | Group name string | Links a checkbox to a named group. JavaScript uses this to coordinate parent-child state. |
| `data-tui-checkbox-parent` | `<input>` | `"true"` | Marks the checkbox as the parent of its group. Toggling the parent checks/unchecks all children. Children update the parent to checked, unchecked, or indeterminate based on their collective state. |

## Usage

```templ
package pages

import "github.com/templui/templui/internal/components/checkbox"

templ PermissionsForm() {
	<form method="post" action="/settings/permissions">
		<fieldset>
			<legend>Notifications</legend>

			<label class="flex items-center gap-2">
				@checkbox.Checkbox(checkbox.Props{
					Name:        "notifications[]",
					Value:       "email",
					Checked:     true,
					Group:       "notifications",
					GroupParent: true,
				})
				Select all
			</label>

			<label class="flex items-center gap-2">
				@checkbox.Checkbox(checkbox.Props{
					Name:  "notifications[]",
					Value: "email",
					Group: "notifications",
				})
				Email notifications
			</label>

			<label class="flex items-center gap-2">
				@checkbox.Checkbox(checkbox.Props{
					Name:  "notifications[]",
					Value: "sms",
					Group: "notifications",
				})
				SMS notifications
			</label>
		</fieldset>

		<button type="submit">Save</button>
	</form>
}
```
