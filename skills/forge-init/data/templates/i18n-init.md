# i18n Init Templates

These files are generated when `i18n` is selected.

---

## i18n/translations/embed.go

```go
package translations

import "embed"

//go:embed */**.json
var FS embed.FS
```

This embed package exposes the translation JSON files as `embed.FS`. Import it from main.go as:

```go
i18ntranslations "{{MODULE_PATH}}/i18n/translations"
```

Then pass `i18ntranslations.FS` to `i18n.WithJSONDir()`.

---

## i18n/translations/en/app.json

```json
{
  "welcome": "Welcome to {{APP_NAME}}",
  "goodbye": "Goodbye!"
}
```

Replace `{{APP_NAME}}` with the actual app name.

This is a starter translation file. The "app" namespace matches the `middlewares.I18nConfig{Namespace: "app"}` setting.

### Translation File Convention

- Directory: `i18n/translations/{lang}/` (e.g., `en/`, `es/`, `fr/`)
- File name: `{namespace}.json` (e.g., `app.json`, `emails.json`)
- Keys can be nested: `{"buttons": {"save": "Save"}}` → accessed as `c.T("buttons.save")`
- Pluralization: `{"items": {"zero": "No items", "one": "1 item", "other": "{{count}} items"}}`

---

## Notes

- The `//go:embed */**.json` directive embeds all `.json` files in all subdirectories.
- Adding a new language is as simple as creating a new directory with translation files — no code changes needed.
- The embed package follows the same pattern as `db/migrations/embed.go` and `assets/embed.go`.
