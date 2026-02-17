# TailwindCSS Subsystem

TailwindCSS is a **CSS build tool** (standalone CLI). When `templ` is NOT selected, it also downloads `@tailwindplus/elements` for interactive JS components. It does NOT add any Go code, config fields, or embed directives on its own — the shared frontend concern (static embed + `forge.WithStaticFiles`) is handled once for all frontend options.

## Imports (config.go)

None.

## Imports (main.go)

None.

## Config Field

None.

## Init Code

None.

## Health Check

None.

## App Option

None — static file serving is handled by the shared frontend concern (see `subsystem-deps.md` resolution rule #3).

## Shutdown Hook

None.

## Env Vars (dev)

None.

## Env Vars (example)

None.

## Docker Service

None.

## Download Commands

**Only when `templ` is NOT selected:** Download the `@tailwindplus/elements` interactive JS library into the vendored assets directory. When `templ` is enabled, skip this — templui provides its own interactive JS components via `Script()` tags.

```bash
curl -sL "https://cdn.jsdelivr.net/npm/@tailwindplus/elements@1" | sed '/^\/\/# sourceMappingURL=/d' > "assets/static/js/elements.min.js"
```

These commands are included in the `assets-download` justfile recipe.

## CSS Source Files

When `tailwind` is selected, generate CSS source file(s) in `assets/src/`. The number of files depends on the run mode:

- **Single-domain:** Create `assets/src/app.css`
- **Multi-domain:** Create one CSS file per domain/router (e.g., `assets/src/website.css`, `assets/src/app.css`)

### CSS Source File Template

Each CSS source file follows this template. Adjust `@source` directives to point to the relevant template/component directories for that domain:

```css
@import "tailwindcss" source(none);
@source "../../components";
@source "../../templates";

@theme {
    --font-sans: "Inter", system-ui, sans-serif;
}
```

When `htmx` is also selected, append HTMX swap transition utilities:

```css
/* HTMX swap transitions */
.htmx-swapping {
    @apply opacity-0 transition-opacity duration-150 ease-out;
}
.htmx-settling {
    @apply opacity-100 transition-opacity duration-150 ease-in;
}
```

For multi-domain setups, each CSS source file should use `@source` directives pointing to that domain's specific template directories (e.g., `@source "../../templates/website"` for the website router).

## Build Commands

CSS compilation is handled by the `css` justfile recipe using the `tailwindcss` standalone CLI binary:

- **Single-domain:** `tailwindcss -i assets/src/app.css -o assets/static/css/app.css --minify`
- **Multi-domain:** One command per domain CSS file

## Notes

- When not using templui, include elements.js via `<script src="/static/js/elements.min.js" type="module"></script>` in your HTML templates.
- Include compiled CSS via `<link rel="stylesheet" href="/static/css/app.css"/>` (or the domain-specific CSS file).
- Requires `tailwindcss` standalone CLI binary installed on the system.
- The `@source` directives tell Tailwind where to scan for utility class usage.
- `source(none)` disables automatic source detection — only explicitly listed `@source` directories are scanned.
