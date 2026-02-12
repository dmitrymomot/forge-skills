# DatePicker

> Date input with a calendar popover, locale-aware formatting, and hidden form field.

## Install

```bash
templui add datepicker
```

Dependencies auto-installed: button, calendar, card, icon, popover

Add to your base layout `<head>`: `@datepicker.Script()`

## Update

```bash
templui add datepicker
```

To update all installed components:

```bash
templui --installed add
```

## API

### Enums

```go
type Format string

const (
	FormatLOCALE_SHORT  Format = "locale-short"  // Locale-specific short format (e.g., MM/DD/YY or DD.MM.YY)
	FormatLOCALE_MEDIUM Format = "locale-medium" // Locale-specific medium format (e.g., Jan 5, 2024 or 5. Jan. 2024)
	FormatLOCALE_LONG   Format = "locale-long"   // Locale-specific long format (e.g., January 5, 2024 or 5. Januar 2024)
	FormatLOCALE_FULL   Format = "locale-full"   // Locale-specific full format (e.g., Monday, January 5, 2024 or Montag, 5. Januar 2024)
)

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
```

### Props

```go
type Props struct {
	ID          string
	Class       string
	Attributes  templ.Attributes
	Name        string
	Value       time.Time
	Form        string
	Format      Format        // Controls the display format using Intl dateStyle options.
	LocaleTag   LocaleTag     // BCP 47 Locale Tag (e.g., "en-US", "es-ES"). Determines language and regional format defaults.
	StartOfWeek *calendar.Day // Optional: 0-6 [Sun-Sat] (Default: 1).
	Placeholder string
	Disabled    bool
	HasError    bool
}
```

## Components

| Function | Props | Description |
| -------- | ----- | ----------- |
| `DatePicker(props ...Props)` | `Props` | Calendar-backed date input with popover, hidden form field, and locale formatting. |

## Composition

```
DatePicker
├── <input type="hidden">          (form submission value, ISO 8601)
├── popover.Trigger
│   └── button.Button              (outline variant, displays formatted date or placeholder)
│       ├── <span>                  (date display text)
│       └── icon.Calendar
└── popover.Content
    └── card.Card
        └── card.Content
            └── calendar.Calendar  (interactive month grid)
```

## Dependencies

- `button` — trigger button (outline variant)
- `calendar` — interactive calendar grid inside popover; also provides `calendar.Day` type used by `StartOfWeek`
- `card` — wrapper inside the popover content
- `icon` — calendar icon on the trigger button
- `popover` — dropdown container for the calendar
- `utils` — class merging, random ID, script URL

## Context

No context passing.

## Data Attributes

| Attribute | Element | Purpose |
| --------- | ------- | ------- |
| `data-tui-datepicker="true"` | trigger button | Identifies the datepicker trigger for JS initialization |
| `data-tui-datepicker-display-format` | trigger button | Display format string (e.g., `"locale-medium"`) |
| `data-tui-datepicker-locale-tag` | trigger button | BCP 47 locale tag (e.g., `"en-US"`) |
| `data-tui-datepicker-placeholder` | trigger button | Placeholder text when no date is selected |
| `data-tui-datepicker-hidden-input` | hidden input | Marks the hidden input that carries the ISO date value |
| `data-tui-datepicker-display` | display span | Marks the span that shows the formatted date or placeholder |

## Usage

```templ
import (
	"github.com/templui/templui/components/datepicker"
	"github.com/templui/templui/components/calendar"
	"time"
)

// Basic usage with defaults
@datepicker.DatePicker()

// With pre-selected date and German locale
@datepicker.DatePicker(datepicker.Props{
	Name:        "birthday",
	Value:       time.Date(1990, 6, 15, 0, 0, 0, 0, time.UTC),
	Format:      datepicker.FormatLOCALE_LONG,
	LocaleTag:   datepicker.LocaleTagGerman,
	Placeholder: "Datum auswahlen",
})
```
