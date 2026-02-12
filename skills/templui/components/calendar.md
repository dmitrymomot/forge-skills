# Calendar

> Interactive date picker calendar with locale support, configurable start-of-week, month/year navigation, and hidden input for form submission.

## Install

```bash
templui add calendar
```

Dependencies auto-installed: icon

Add to your base layout `<head>`:

```go
@calendar.Script()
```

## Update

```bash
templui add calendar
```

To update all installed components:

```bash
templui --installed add
```

## API

### Enums

```go
type LocaleTag string

var (
	LocaleDefaultTag    = LocaleTag("en-US")
	LocaleTagChinese    = LocaleTag("zh-CN")
	LocaleTagFrench     = LocaleTag("fr-FR")
	LocaleTagGerman     = LocaleTag("de-DE")
	LocaleTagItalian    = LocaleTag("it-IT")
	LocaleTagJapanese   = LocaleTag("ja-JP")
	LocaleTagPortuguese = LocaleTag("pt-PT")
	LocaleTagSpanish    = LocaleTag("es-ES")
)

type Day int

var (
	Sunday    = Day(0)
	Monday    = Day(1)
	Tuesday   = Day(2)
	Wednesday = Day(3)
	Thursday  = Day(4)
	Friday    = Day(5)
	Saturday  = Day(6)
)
```

### Props

```go
type Props struct {
	ID                string
	Class             string
	LocaleTag         LocaleTag
	Value             *time.Time
	Name              string
	InitialMonth      int  // Optional: 0-11 (Default: current or from Value). Controls the initially displayed month view.
	InitialYear       int  // Optional: (Default: current or from Value). Controls the initially displayed year view.
	StartOfWeek       *Day // Optional: 0-6 [Sun-Sat] (Default: 1).
	RenderHiddenInput bool // Optional: Whether to render the hidden input (Default: true). Set to false when used inside DatePicker.
}
```

| Prop | Type | Default | Description |
| ---- | ---- | ------- | ----------- |
| ID | `string` | `""` (auto-generated) | HTML `id` for the calendar container. Auto-generated with `"-calendar"` suffix when empty. |
| Class | `string` | `""` | Additional CSS classes applied to the outer wrapper `<div>`. |
| LocaleTag | `LocaleTag` | `LocaleDefaultTag` (`"en-US"`) | Locale tag for localized month and weekday names via `Intl.DateTimeFormat`. |
| Value | `*time.Time` | `nil` | Pre-selected date. When set, the calendar opens to that date's month/year and highlights the day. Formatted as ISO `"2006-01-02"` in the hidden input. |
| Name | `string` | `"{ID}-value"` | HTML `name` attribute on the hidden `<input>`. Used for form submission. |
| InitialMonth | `int` | current month or from Value | Initial month view (0-11). Overrides the month derived from `Value` or the current date. |
| InitialYear | `int` | current year or from Value | Initial year view. Overrides the year derived from `Value` or the current date. |
| StartOfWeek | `*Day` | `Monday` (`Day(1)`) | First day of the week in the grid. Pointer type; `nil` defaults to Monday. |
| RenderHiddenInput | `bool` | `true` | Whether to render a hidden `<input>` for form submission. Set to `false` when the calendar is embedded inside a parent component (e.g., DatePicker) that manages its own input. |

## Components

| Function | Props | Description |
| -------- | ----- | ----------- |
| `calendar.Calendar(props ...Props)` | `calendar.Props` | Renders a full calendar widget with month/year selects, prev/next navigation arrows, weekday headers, and a day grid. Includes a hidden input for form value binding. |

## Composition

```
Calendar (wrapper div)
  <input type="hidden">           (form value, conditional on RenderHiddenInput)
  Calendar Container
    Header
      Prev Button (icon.ChevronLeft)
      Month Select (native <select> with visual overlay)
        icon.ChevronDown
      Year Select (native <select> with visual overlay)
        icon.ChevronDown
      Next Button (icon.ChevronRight)
    Weekday Headers Grid          (populated by JS)
    Day Grid                      (populated by JS)
```

## Dependencies

- `icon` -- provides `ChevronLeft`, `ChevronRight` (navigation arrows) and `ChevronDown` (select indicators).
- `utils` -- `RandomID` for auto-generating element IDs.

## Context

No context passing.

## Data Attributes

| Attribute | Set On | Value | Description |
| --------- | ------ | ----- | ----------- |
| `data-tui-calendar-wrapper` | Outer wrapper `<div>` | `"true"` | Identifies the calendar wrapper for locating the hidden input. |
| `data-tui-calendar-hidden-input` | Hidden `<input>` | (presence) | Marks the hidden input element. JS updates its value on date selection. |
| `data-tui-calendar-container` | Calendar container `<div>` | `"true"` | Main calendar container. JS attaches all behavior to elements within this container. |
| `data-tui-calendar-locale-tag` | Calendar container | Locale string (e.g. `"en-US"`) | Locale used by JS for `Intl.DateTimeFormat` formatting of month and weekday names. |
| `data-tui-calendar-initial-month` | Calendar container | `"0"`-`"11"` | Server-rendered initial month (0-indexed). Used by JS on first render. |
| `data-tui-calendar-initial-year` | Calendar container | Year string (e.g. `"2025"`) | Server-rendered initial year. Used by JS on first render. |
| `data-tui-calendar-selected-date` | Calendar container | ISO date string (e.g. `"2025-06-15"`) | Currently selected date. Updated by JS on day click. |
| `data-tui-calendar-start-of-week` | Calendar container | `"0"`-`"6"` | Start-of-week day index (0=Sunday, 1=Monday, etc.). |
| `data-tui-calendar-prev` | Prev button | (presence) | Identifies the previous-month navigation button. |
| `data-tui-calendar-next` | Next button | (presence) | Identifies the next-month navigation button. |
| `data-tui-calendar-month-select` | Month `<select>` | (presence) | Identifies the native month select element. |
| `data-tui-calendar-year-select` | Year `<select>` | (presence) | Identifies the native year select element. |
| `data-tui-calendar-weekdays` | Weekday header grid `<div>` | (presence) | Container for localized weekday abbreviations, populated by JS. |
| `data-tui-calendar-days` | Day grid `<div>` | (presence) | Container for day buttons, populated by JS. |
| `data-tui-calendar-day` | Day `<button>` (JS-generated) | Day number string (e.g. `"15"`) | Identifies each day button. Set by JS during grid rendering. |
| `data-tui-calendar-today` | Day `<button>` (JS-generated) | `"true"` | Marks today's date button. Set by JS during grid rendering. |
| `data-tui-calendar-selected` | Day `<button>` (JS-generated) | `"true"` | Marks the currently selected date button. Set by JS during grid rendering. |

## Usage

```templ
package pages

import (
	"time"
	"github.com/templui/templui/internal/components/calendar"
)

templ BookingForm() {
	<form method="post" action="/bookings">
		<label class="block mb-2 font-medium">Select a date</label>
		@calendar.Calendar(calendar.Props{
			Name:        "booking_date",
			LocaleTag:   calendar.LocaleDefaultTag,
			StartOfWeek: &calendar.Monday,
		})
		<button type="submit" class="mt-4">Book</button>
	</form>
}
```
