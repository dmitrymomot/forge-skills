# TimePicker

> Time picker component with hour/minute selection grid, optional 12-hour format with AM/PM toggle, min/max time constraints, and popover-based UI.

## Install

```bash
templui add timepicker
```

Dependencies auto-installed: button, card, icon, popover

Add to your base layout `<head>`:

```go
@timepicker.Script()
```

## Update

```bash
templui add timepicker
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
	Form        string
	Value       time.Time
	MinTime     time.Time
	MaxTime     time.Time
	Step        int
	Use12Hours  bool
	AMLabel     string
	PMLabel     string
	Placeholder string
	Disabled    bool
	HasError    bool
}
```

| Prop | Type | Default | Description |
| ---- | ---- | ------- | ----------- |
| ID | `string` | `utils.RandomID()` | HTML `id` for the trigger button. Also used to derive the hidden input ID (`ID + "-hidden"`) and popover content ID (`ID + "-content"`). |
| Class | `string` | `""` | Additional CSS classes merged via `utils.TwMerge` onto the trigger button. |
| Attributes | `templ.Attributes` | `nil` | Arbitrary HTML attributes spread onto the trigger button. |
| Name | `string` | same as `ID` | HTML `name` attribute on the hidden `<input>`. Used for form submission. |
| Form | `string` | `""` | Associates the hidden input with a `<form>` by form ID. |
| Value | `time.Time` | zero `time.Time` | Initial selected time. Formatted as `"15:04"` (24-hour) for the hidden input value. |
| MinTime | `time.Time` | zero `time.Time` | Earliest selectable time. Hours/minutes outside this constraint are disabled. |
| MaxTime | `time.Time` | zero `time.Time` | Latest selectable time. Hours/minutes outside this constraint are disabled. |
| Step | `int` | `1` | Minute increment step. Minutes are generated from 0 to 59 in increments of this value. |
| Use12Hours | `bool` | `false` | When true, displays hours 12, 01-11 with AM/PM toggle instead of 00-23. |
| AMLabel | `string` | `"AM"` | Label text for the AM period button (only used when `Use12Hours` is true). |
| PMLabel | `string` | `"PM"` | Label text for the PM period button (only used when `Use12Hours` is true). |
| Placeholder | `string` | `"Select time"` | Placeholder text shown in the trigger button when no time is selected. |
| Disabled | `bool` | `false` | Disables the trigger button, preventing interaction. |
| HasError | `bool` | `false` | Applies error/destructive styling to the trigger button border and ring. Sets `aria-invalid="true"`. |

## Components

| Function | Props | Description |
| -------- | ----- | ----------- |
| `timepicker.TimePicker(props ...Props)` | `timepicker.Props` | Renders a time picker with a hidden input for form submission, a trigger button showing the selected time or placeholder, and a popover containing hour/minute selection grids with an optional AM/PM toggle and Done button. |

## Composition

```
TimePicker
  <input type="hidden">                     (form value, "HH:MM" 24-hour format)
  popover.Trigger
    button.Button                            (trigger, outline variant)
      <span>                                 (display text: selected time or placeholder)
      icon.Clock                             (trailing clock icon, size 16)
  popover.Content                            (placement: BottomStart, width: w-80)
    card.Card
      card.Content
        <div>                                (popup container)
          <div>                              (grid: 2 columns)
            <div>                            (hour column)
              <label>                        ("Hour")
              <div>                          (scrollable hour list)
                <button>...                  (one per hour: 0-23 or 12,01-11)
            <div>                            (minute column)
              <label>                        ("Minute")
              <div>                          (scrollable minute list)
                <button>...                  (one per step: 0, step, 2*step, ...)
          <div>                              (footer row)
            <div>                            (AM/PM buttons, only if Use12Hours)
              <button>                       (AM period)
              <button>                       (PM period)
            button.Button                    (Done, secondary variant, size sm)
```

## Dependencies

- `button` -- trigger button and Done button.
- `card` -- popover content wrapper (`Card` and `Content`).
- `icon` -- `Clock` icon on the trigger button.
- `popover` -- `Trigger` and `Content` for the dropdown panel.
- `utils` -- `TwMerge`, `RandomID`, `If`, `MergeAttributes`.

## Context

No context passing.

## Data Attributes

| Attribute | Set On | Value | Description |
| --------- | ------ | ----- | ----------- |
| `data-tui-timepicker` | trigger `<button>` | `"true"` | Identifies the timepicker trigger element. JavaScript uses this to initialize and manage the component. |
| `data-tui-timepicker-use12hours` | trigger `<button>` | `"true"` / `"false"` | Indicates whether 12-hour format is enabled. |
| `data-tui-timepicker-am-label` | trigger `<button>` | string | The AM label text. |
| `data-tui-timepicker-pm-label` | trigger `<button>` | string | The PM label text. |
| `data-tui-timepicker-placeholder` | trigger `<button>` | string | Placeholder text for the display span. |
| `data-tui-timepicker-step` | trigger `<button>` | integer string | Minute step value. |
| `data-tui-timepicker-min-time` | trigger `<button>` | `"HH:MM"` or `""` | Minimum selectable time in 24-hour format. |
| `data-tui-timepicker-max-time` | trigger `<button>` | `"HH:MM"` or `""` | Maximum selectable time in 24-hour format. |
| `data-tui-timepicker-hidden-input` | hidden `<input>` | `"true"` | Marks the hidden input for JavaScript discovery and reactive binding. |
| `data-tui-timepicker-display` | `<span>` | (none) | Marks the display element inside the trigger that shows the selected time or placeholder text. |
| `data-tui-timepicker-popup` | popup `<div>` | `"true"` | Identifies the popup container inside the popover content. |
| `data-tui-timepicker-input-name` | popup `<div>` | string | The form input name, stored on the popup for JavaScript reference. |
| `data-tui-timepicker-parent-id` | popup `<div>` | string | The trigger button ID, stored on the popup for JavaScript reference. |
| `data-tui-timepicker-value` | popup `<div>` | `"HH:MM"` | Initial time value in 24-hour format (only set when a value is provided). |
| `data-tui-timepicker-hour` | hour `<button>` | integer string | The hour value this button represents (0-23 or 0-11 depending on format). |
| `data-tui-timepicker-minute` | minute `<button>` | integer string | The minute value this button represents. |
| `data-tui-timepicker-selected` | hour/minute `<button>` | `"true"` / `"false"` | Whether this hour or minute button is currently selected. Drives primary highlight styling. |
| `data-tui-timepicker-hour-list` | hour list `<div>` | `"true"` | Marks the hour button container for JavaScript selection management. |
| `data-tui-timepicker-minute-list` | minute list `<div>` | `"true"` | Marks the minute button container for JavaScript selection management. |
| `data-tui-timepicker-period` | AM/PM `<button>` | `"AM"` / `"PM"` | Identifies the AM or PM period toggle button. |
| `data-tui-timepicker-active` | AM/PM `<button>` | `"true"` / `"false"` | Whether this period button is the currently active period. Drives primary highlight styling. |
| `data-tui-timepicker-done` | Done `<button>` | `"true"` | Marks the Done button. Clicking it closes the popover via `window.closePopover`. |

## Usage

```templ
package pages

import (
	"time"
	"github.com/templui/templui/internal/components/timepicker"
)

templ MeetingForm() {
	<form method="post" action="/meetings/create">
		<div class="space-y-4">
			<div>
				<label class="text-sm font-medium">Start Time</label>
				@timepicker.TimePicker(timepicker.Props{
					Name:        "start_time",
					Placeholder: "Pick a start time",
					Step:        15,
					MinTime:     time.Date(0, 1, 1, 9, 0, 0, 0, time.UTC),
					MaxTime:     time.Date(0, 1, 1, 17, 0, 0, 0, time.UTC),
				})
			</div>

			<div>
				<label class="text-sm font-medium">End Time</label>
				@timepicker.TimePicker(timepicker.Props{
					Name:       "end_time",
					Use12Hours: true,
					Step:       30,
				})
			</div>

			<button type="submit">Create Meeting</button>
		</div>
	</form>
}
```
