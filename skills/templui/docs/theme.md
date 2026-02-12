# Tailwind CSS v4 Theme for templui

templui components use CSS custom properties for theming. This guide covers setting up `input.css` with the full theme system.

## Why This Is Needed

- **forge-init** creates basic CSS source files in `assets/src/` with a minimal `@theme` block (font-sans only) — compiled via the `tailwindcss` CLI
- **templui components** reference CSS variables like `bg-background`, `text-primary`, `border-border` — these must be defined via additional CSS custom properties
- The forge-init CSS source files need to be **extended** with templui's theme variables for component theming to work

## input.css

Create `assets/css/input.css` with the following content:

```css
@import "tailwindcss";

@custom-variant dark (&:where(.dark, .dark *));

@theme inline {
  --breakpoint-3xl: 120rem;

  --radius-sm: calc(var(--radius) - 4px);
  --radius-md: calc(var(--radius) - 2px);
  --radius-lg: var(--radius);
  --radius-xl: calc(var(--radius) + 4px);

  --color-background: oklch(var(--background));
  --color-foreground: oklch(var(--foreground));
  --color-card: oklch(var(--card));
  --color-card-foreground: oklch(var(--card-foreground));
  --color-popover: oklch(var(--popover));
  --color-popover-foreground: oklch(var(--popover-foreground));
  --color-primary: oklch(var(--primary));
  --color-primary-foreground: oklch(var(--primary-foreground));
  --color-secondary: oklch(var(--secondary));
  --color-secondary-foreground: oklch(var(--secondary-foreground));
  --color-muted: oklch(var(--muted));
  --color-muted-foreground: oklch(var(--muted-foreground));
  --color-accent: oklch(var(--accent));
  --color-accent-foreground: oklch(var(--accent-foreground));
  --color-destructive: oklch(var(--destructive));
  --color-destructive-foreground: oklch(var(--destructive-foreground));
  --color-border: oklch(var(--border));
  --color-input: oklch(var(--input));
  --color-ring: oklch(var(--ring));
  --color-chart-1: oklch(var(--chart-1));
  --color-chart-2: oklch(var(--chart-2));
  --color-chart-3: oklch(var(--chart-3));
  --color-chart-4: oklch(var(--chart-4));
  --color-chart-5: oklch(var(--chart-5));
  --color-sidebar: oklch(var(--sidebar));
  --color-sidebar-foreground: oklch(var(--sidebar-foreground));
  --color-sidebar-primary: oklch(var(--sidebar-primary));
  --color-sidebar-primary-foreground: oklch(var(--sidebar-primary-foreground));
  --color-sidebar-accent: oklch(var(--sidebar-accent));
  --color-sidebar-accent-foreground: oklch(var(--sidebar-accent-foreground));
  --color-sidebar-border: oklch(var(--sidebar-border));
  --color-sidebar-ring: oklch(var(--sidebar-ring));
}

:root {
  --radius: 0.65rem;
  --background: 1 0 0;
  --foreground: 0.141 0.005 285.823;
  --card: 1 0 0;
  --card-foreground: 0.141 0.005 285.823;
  --popover: 1 0 0;
  --popover-foreground: 0.141 0.005 285.823;
  --primary: 0.21 0.006 285.885;
  --primary-foreground: 0.985 0 0;
  --secondary: 0.967 0.001 286.375;
  --secondary-foreground: 0.21 0.006 285.885;
  --muted: 0.967 0.001 286.375;
  --muted-foreground: 0.552 0.016 285.938;
  --accent: 0.967 0.001 286.375;
  --accent-foreground: 0.21 0.006 285.885;
  --destructive: 0.577 0.245 27.325;
  --destructive-foreground: 0.985 0 0;
  --border: 0.92 0.004 286.32;
  --input: 0.92 0.004 286.32;
  --ring: 0.705 0.015 286.067;
  --chart-1: 0.646 0.222 41.116;
  --chart-2: 0.6 0.118 184.714;
  --chart-3: 0.398 0.07 227.392;
  --chart-4: 0.828 0.189 84.429;
  --chart-5: 0.769 0.188 70.08;
  --sidebar: 0.985 0 0;
  --sidebar-foreground: 0.141 0.005 285.823;
  --sidebar-primary: 0.21 0.006 285.885;
  --sidebar-primary-foreground: 0.985 0 0;
  --sidebar-accent: 0.967 0.001 286.375;
  --sidebar-accent-foreground: 0.21 0.006 285.885;
  --sidebar-border: 0.92 0.004 286.32;
  --sidebar-ring: 0.705 0.015 286.067;
}

.dark {
  --background: 0.141 0.005 285.823;
  --foreground: 0.985 0 0;
  --card: 0.141 0.005 285.823;
  --card-foreground: 0.985 0 0;
  --popover: 0.141 0.005 285.823;
  --popover-foreground: 0.985 0 0;
  --primary: 0.985 0 0;
  --primary-foreground: 0.21 0.006 285.885;
  --secondary: 0.274 0.006 286.033;
  --secondary-foreground: 0.985 0 0;
  --muted: 0.274 0.006 286.033;
  --muted-foreground: 0.705 0.015 286.067;
  --accent: 0.274 0.006 286.033;
  --accent-foreground: 0.985 0 0;
  --destructive: 0.396 0.141 25.768;
  --destructive-foreground: 0.985 0 0;
  --border: 0.274 0.006 286.033;
  --input: 0.274 0.006 286.033;
  --ring: 0.442 0.017 285.786;
  --chart-1: 0.488 0.243 264.376;
  --chart-2: 0.696 0.17 162.48;
  --chart-3: 0.769 0.188 70.08;
  --chart-4: 0.627 0.265 303.9;
  --chart-5: 0.645 0.246 16.439;
  --sidebar: 0.141 0.005 285.823;
  --sidebar-foreground: 0.985 0 0;
  --sidebar-primary: 0.488 0.243 264.376;
  --sidebar-primary-foreground: 0.985 0 0;
  --sidebar-accent: 0.274 0.006 286.033;
  --sidebar-accent-foreground: 0.985 0 0;
  --sidebar-border: 0.274 0.006 286.033;
  --sidebar-ring: 0.442 0.017 285.786;
}

@layer base {
  * {
    @apply border-border;
  }

  *::-webkit-scrollbar {
    @apply h-1.5 w-1.5;
  }

  *::-webkit-scrollbar-thumb {
    @apply rounded-full bg-muted-foreground/30;
  }

  *::-webkit-scrollbar-track {
    @apply bg-transparent;
  }

  body {
    @apply bg-background text-foreground;
  }
}
```

## CSS Variable Reference

All color values use OKLCH format (`lightness chroma hue`).

### Core Variables

| Variable                   | Purpose                                        |
| -------------------------- | ---------------------------------------------- |
| `--background`             | Page/app background                            |
| `--foreground`             | Default text color                             |
| `--card`                   | Card surface color                             |
| `--card-foreground`        | Card text color                                |
| `--popover`                | Popover/dropdown surface                       |
| `--popover-foreground`     | Popover text color                             |
| `--primary`                | Primary buttons, links, accents                |
| `--primary-foreground`     | Text on primary surfaces                       |
| `--secondary`              | Secondary buttons, less emphasis               |
| `--secondary-foreground`   | Text on secondary surfaces                     |
| `--muted`                  | Muted backgrounds (subtle UI)                  |
| `--muted-foreground`       | Muted/placeholder text                         |
| `--accent`                 | Accent highlights (hover states, active items) |
| `--accent-foreground`      | Text on accent surfaces                        |
| `--destructive`            | Destructive/error actions                      |
| `--destructive-foreground` | Text on destructive surfaces                   |
| `--border`                 | Default border color                           |
| `--input`                  | Input border color                             |
| `--ring`                   | Focus ring color                               |

### Chart Variables

| Variable                        | Purpose                   |
| ------------------------------- | ------------------------- |
| `--chart-1` through `--chart-5` | Chart/graph color palette |

### Sidebar Variables

| Variable                       | Purpose                       |
| ------------------------------ | ----------------------------- |
| `--sidebar`                    | Sidebar background            |
| `--sidebar-foreground`         | Sidebar text                  |
| `--sidebar-primary`            | Sidebar active/primary items  |
| `--sidebar-primary-foreground` | Text on sidebar primary items |
| `--sidebar-accent`             | Sidebar hover/accent state    |
| `--sidebar-accent-foreground`  | Text on sidebar accent items  |
| `--sidebar-border`             | Sidebar border color          |
| `--sidebar-ring`               | Sidebar focus ring            |

### Radius System

The radius system uses a single base value with computed variants:

```
--radius: 0.65rem          (base)
--radius-sm: radius - 4px  (small elements like checkboxes)
--radius-md: radius - 2px  (inputs, buttons)
--radius-lg: radius        (cards, dialogs)
--radius-xl: radius + 4px  (large containers)
```

## Dark Mode Toggle

Add this script to your base layout for system-preference detection and manual toggle:

```html
<script>
  // Apply theme immediately to prevent flash
  (function () {
    const saved = localStorage.getItem("theme");
    if (saved === "dark" || (!saved && window.matchMedia("(prefers-color-scheme: dark)").matches)) {
      document.documentElement.classList.add("dark");
    }
  })();

  function cycleTheme() {
    const html = document.documentElement;
    const isDark = html.classList.contains("dark");
    if (isDark) {
      html.classList.remove("dark");
      localStorage.setItem("theme", "light");
    } else {
      html.classList.add("dark");
      localStorage.setItem("theme", "dark");
    }
  }
</script>
```

## Extending the Forge-Init CSS

forge-init already creates CSS source files in `assets/src/` and a `css` Taskfile task that compiles them with the `tailwindcss` standalone CLI. To add templui theme support, extend the existing CSS source file(s).

### 1. Update CSS Source File

Replace the content of `assets/src/app.css` (or the domain-specific CSS file) with the full `input.css` content shown above. This adds the templui theme variables (`--background`, `--primary`, etc.) and the `@custom-variant dark` directive on top of the existing Tailwind setup.

Keep the existing `@source` directives from forge-init but add the `@custom-variant`, `@theme inline`, `:root`, `.dark`, and `@layer base` sections from this file.

### 2. Rebuild CSS

Run the existing `css` Taskfile task to recompile:

```bash
go tool task css
```

### 3. Watch Mode (Optional)

For development, you can run the `tailwindcss` CLI in watch mode:

```bash
tailwindcss -i assets/src/app.css -o assets/static/css/app.css --watch
```

Consider adding a `css:watch` task to your `Taskfile.yml` for convenience.
