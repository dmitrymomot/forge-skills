# Sidebar

> Collapsible sidebar component for app layouts with mobile sheet support, icon-collapse mode, and keyboard shortcuts.

## Install

```bash
templui add sidebar
```

Dependencies auto-installed: button, icon, sheet, tooltip

Add to your base layout `<head>`: `@sidebar.Script()`

## Update

```bash
templui add sidebar
```

To update all: `templui --installed add`

## API

### Enums

```go
type Side string

const (
	SideLeft  Side = "left"  // default
	SideRight Side = "right"
)

type Variant string

const (
	VariantSidebar  Variant = "sidebar"  // default
	VariantFloating Variant = "floating"
	VariantInset    Variant = "inset"
)

type Collapsible string

const (
	CollapsibleOffcanvas Collapsible = "offcanvas" // default
	CollapsibleIcon      Collapsible = "icon"
	CollapsibleNone      Collapsible = "none"
)

type MenuButtonSize string

const (
	MenuButtonSizeDefault MenuButtonSize = "default" // default
	MenuButtonSizeSm      MenuButtonSize = "sm"
	MenuButtonSizeLg      MenuButtonSize = "lg"
)
```

### Props

```go
type LayoutProps struct {
	ID         string
	Class      string
	Attributes templ.Attributes
}

type Props struct {
	ID               string
	Class            string
	Attributes       templ.Attributes
	Side             Side        // default: "left"
	Variant          Variant     // default: "sidebar"
	Collapsible      Collapsible // default: "offcanvas"
	Collapsed        bool        // default: false (sidebar open)
	KeyboardShortcut string      // default: "b"
}

type TriggerProps struct {
	ID         string
	Class      string
	Attributes templ.Attributes
	Target     string // Target sidebar ID to toggle
}

type HeaderProps struct {
	ID         string
	Class      string
	Attributes templ.Attributes
}

type ContentProps struct {
	ID         string
	Class      string
	Attributes templ.Attributes
}

type FooterProps struct {
	ID         string
	Class      string
	Attributes templ.Attributes
}

type InsetProps struct {
	ID         string
	Class      string
	Attributes templ.Attributes
}

type GroupProps struct {
	ID         string
	Class      string
	Attributes templ.Attributes
}

type GroupLabelProps struct {
	ID         string
	Class      string
	Attributes templ.Attributes
}

type MenuProps struct {
	ID         string
	Class      string
	Attributes templ.Attributes
}

type MenuItemProps struct {
	ID         string
	Class      string
	Attributes templ.Attributes
}

type MenuButtonProps struct {
	ID         string
	Class      string
	Attributes templ.Attributes
	Href       string
	IsActive   bool
	Size       MenuButtonSize // default: "default"
	Tooltip    string         // Tooltip text to show when sidebar is collapsed
}

type MenuBadgeProps struct {
	ID         string
	Class      string
	Attributes templ.Attributes
}

type MenuSubProps struct {
	ID         string
	Class      string
	Attributes templ.Attributes
}

type MenuSubItemProps struct {
	ID         string
	Class      string
	Attributes templ.Attributes
}

type MenuSubButtonProps struct {
	ID         string
	Class      string
	Attributes templ.Attributes
	Href       string
	IsActive   bool
}

type SeparatorProps struct {
	ID         string
	Class      string
	Attributes templ.Attributes
}
```

## Components

| Function | Props | Description |
| -------- | ----- | ----------- |
| `Layout(props ...LayoutProps)` | `LayoutProps` | Top-level flex container wrapping the sidebar and main content. Sets sidebar ID in context. |
| `Sidebar(props ...Props)` | `Props` | The sidebar itself. Renders a mobile sheet (via `sheet` component) for small screens and a fixed `<aside>` for desktop. Reads ID from context if not provided. |
| `Trigger(props ...TriggerProps)` | `TriggerProps` | Toggle button for the sidebar. Renders a sheet trigger on mobile and a desktop trigger with `data-tui-sidebar-trigger`. Reads target ID from context if `Target` is empty. |
| `Header(props ...HeaderProps)` | `HeaderProps` | Top section inside the sidebar (e.g., logo, brand). |
| `Content(props ...ContentProps)` | `ContentProps` | Scrollable middle section of the sidebar that holds navigation groups. |
| `Footer(props ...FooterProps)` | `FooterProps` | Bottom section pinned via `mt-auto` (e.g., user menu, settings). |
| `Inset(props ...InsetProps)` | `InsetProps` | Main content area rendered as `<main>`. Receives inset styling when the peer sidebar uses `VariantInset`. |
| `Group(props ...GroupProps)` | `GroupProps` | Groups related menu items together with padding. |
| `GroupLabel(props ...GroupLabelProps)` | `GroupLabelProps` | Label heading for a `Group`. Hides when sidebar is collapsed to icon mode. |
| `Menu(props ...MenuProps)` | `MenuProps` | Navigation list (`<ul>`) containing `MenuItem` children. |
| `MenuItem(props ...MenuItemProps)` | `MenuItemProps` | Single list item (`<li>`) wrapping a `MenuButton`. |
| `MenuButton(props ...MenuButtonProps)` | `MenuButtonProps` | Navigation link or button. Renders `<a>` when `Href` is set, `<button>` otherwise. Shows tooltip when collapsed to icon mode if `Tooltip` is provided. |
| `MenuBadge(props ...MenuBadgeProps)` | `MenuBadgeProps` | Badge displayed to the right of a menu button label. Hidden in icon-collapse mode. |
| `MenuSub(props ...MenuSubProps)` | `MenuSubProps` | Nested sub-menu list (`<ul>`) with left border. Hidden in icon-collapse mode. |
| `MenuSubItem(props ...MenuSubItemProps)` | `MenuSubItemProps` | List item (`<li>`) inside a `MenuSub`. |
| `MenuSubButton(props ...MenuSubButtonProps)` | `MenuSubButtonProps` | Link or button inside a `MenuSubItem`. Renders `<a>` when `Href` is set. |
| `Separator(props ...SeparatorProps)` | `SeparatorProps` | Horizontal rule divider between sidebar sections. |

## Composition

```
Layout
  Sidebar
    Header
      ...branding content...
    Content
      Group
        GroupLabel
        Menu
          MenuItem
            MenuButton
            MenuBadge
          MenuItem
            MenuButton
            MenuSub
              MenuSubItem
                MenuSubButton
      Separator
      Group
        ...
    Footer
      ...user/settings content...
  Inset
    ...main page content...
    Trigger  (can be placed anywhere in Inset)
```

## Dependencies

- `button` -- used internally by `Trigger` for the toggle button
- `icon` -- `icon.PanelLeft` rendered inside the trigger button
- `sheet` -- mobile sidebar rendered as a sheet overlay on screens < 768px
- `tooltip` -- optional tooltip on `MenuButton` when sidebar is collapsed to icon mode

## Context

The `Layout` component stores a sidebar ID in context via the key `sidebarIDKey` (type `contextKey("sidebar-id")`). Both `Sidebar` and `Trigger` read this value so they automatically target the correct sidebar without explicit ID wiring.

## Data Attributes

| Attribute | Element | Values / Purpose |
| --------- | ------- | ---------------- |
| `data-tui-sidebar-layout` | `Layout` wrapper div | Marks the layout container. |
| `data-tui-sidebar-wrapper` | `Sidebar` outer div | Marks the sidebar wrapper for JS targeting. |
| `data-tui-sidebar-id` | `Sidebar` outer div | The sidebar's unique ID. |
| `data-tui-sidebar-state` | `Sidebar` outer div | `"expanded"` or `"collapsed"`. Toggled by JS. |
| `data-tui-sidebar-collapsible` | `Sidebar` outer div | `"offcanvas"`, `"icon"`, or `"none"`. |
| `data-tui-sidebar-variant` | `Sidebar` outer div | `"sidebar"`, `"floating"`, or `"inset"`. |
| `data-tui-sidebar-side` | `Sidebar` outer div | `"left"` or `"right"`. |
| `data-tui-sidebar-keyboard-shortcut` | `Sidebar` outer div | Keyboard shortcut key (default `"b"`). Ctrl/Cmd + key toggles sidebar. |
| `data-tui-sidebar-content` | Inner content div | Sidebar ID. Used by JS to portal content between desktop and mobile. |
| `data-tui-sidebar-mobile-portal` | Mobile portal div | Sidebar ID. Target for mobile content portaling. |
| `data-tui-sidebar-trigger` | Desktop trigger button | Marks the trigger for JS click handling. |
| `data-tui-sidebar-target` | Desktop trigger button | ID of the sidebar to toggle. |
| `data-tui-sidebar="header"` | `Header` | Semantic marker. |
| `data-tui-sidebar="content"` | `Content` | Semantic marker. |
| `data-tui-sidebar="footer"` | `Footer` | Semantic marker. |
| `data-tui-sidebar="inset"` | `Inset` | Semantic marker. |
| `data-tui-sidebar="group"` | `Group` | Semantic marker. |
| `data-tui-sidebar="group-label"` | `GroupLabel` | Semantic marker. |
| `data-tui-sidebar="menu"` | `Menu` | Semantic marker. |
| `data-tui-sidebar="menu-item"` | `MenuItem` | Semantic marker. |
| `data-tui-sidebar="menu-button"` | `MenuButton` | Semantic marker. |
| `data-tui-sidebar-size` | `MenuButton` | `"default"`, `"sm"`, or `"lg"`. Controls height/text size. |
| `data-tui-sidebar-active` | `MenuButton`, `MenuSubButton` | `"true"` when active. Applies accent background and font-medium. |
| `data-tui-sidebar="menu-badge"` | `MenuBadge` | Semantic marker. |
| `data-tui-sidebar="menu-sub"` | `MenuSub` | Semantic marker. |
| `data-tui-sidebar="menu-sub-item"` | `MenuSubItem` | Semantic marker. |
| `data-tui-sidebar="menu-sub-button"` | `MenuSubButton` | Semantic marker. |
| `data-tui-sidebar="separator"` | `Separator` | Semantic marker. |

## Usage

```templ
package pages

import "yourapp/components/sidebar"
import "yourapp/components/icon"

templ DashboardLayout() {
	@sidebar.Layout() {
		@sidebar.Sidebar(sidebar.Props{
			Variant:     sidebar.VariantSidebar,
			Collapsible: sidebar.CollapsibleIcon,
		}) {
			@sidebar.Header() {
				<span class="text-lg font-semibold">Acme Inc</span>
			}
			@sidebar.Content() {
				@sidebar.Group() {
					@sidebar.GroupLabel() {
						Platform
					}
					@sidebar.Menu() {
						@sidebar.MenuItem() {
							@sidebar.MenuButton(sidebar.MenuButtonProps{
								Href:     "/dashboard",
								IsActive: true,
								Tooltip:  "Dashboard",
							}) {
								@icon.LayoutDashboard(icon.Props{})
								<span>Dashboard</span>
							}
						}
						@sidebar.MenuItem() {
							@sidebar.MenuButton(sidebar.MenuButtonProps{
								Href:    "/settings",
								Tooltip: "Settings",
							}) {
								@icon.Settings(icon.Props{})
								<span>Settings</span>
								@sidebar.MenuBadge() {
									3
								}
							}
						}
					}
				}
				@sidebar.Separator()
				@sidebar.Group() {
					@sidebar.GroupLabel() {
						Projects
					}
					@sidebar.Menu() {
						@sidebar.MenuItem() {
							@sidebar.MenuButton(sidebar.MenuButtonProps{Href: "/projects"}) {
								@icon.Folder(icon.Props{})
								<span>All Projects</span>
							}
							@sidebar.MenuSub() {
								@sidebar.MenuSubItem() {
									@sidebar.MenuSubButton(sidebar.MenuSubButtonProps{
										Href: "/projects/alpha",
									}) {
										Alpha
									}
								}
							}
						}
					}
				}
			}
			@sidebar.Footer() {
				@sidebar.Menu() {
					@sidebar.MenuItem() {
						@sidebar.MenuButton(sidebar.MenuButtonProps{
							Href: "/account",
							Size: sidebar.MenuButtonSizeLg,
						}) {
							<span>Jane Doe</span>
						}
					}
				}
			}
		}
		@sidebar.Inset() {
			<header class="flex items-center gap-2 p-4 border-b">
				@sidebar.Trigger()
				<h1>Dashboard</h1>
			</header>
			<div class="p-6">
				Main content goes here.
			</div>
		}
	}
}
```
