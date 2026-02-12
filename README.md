# forge-skills

Claude Code plugin for building applications with the [Forge](https://github.com/dmitrymomot/forge) Go micro-SaaS framework.

## Installation

Inside Claude Code, run:

```
/plugin marketplace add dmitrymomot/forge-skills
/plugin install forge-skills@forge-skills
```

Requires Claude Code v1.0.33+.

## Skills

### `/forge-init <app-name>`

Scaffolds a complete Forge project with configurable subsystems: PostgreSQL, Redis, sessions, jobs, storage, templ, HTMX, Alpine.js, Tailwind CSS, mailer, and OAuth.

Generates: `main.go`, config, `docker-compose.yml`, `.env`, `Taskfile.yml`, `.air.toml`, `.editorconfig`, `.gitignore`.

### `/forge-build <feature description>`

Builds features into existing Forge apps. Generates handlers, DB migrations + sqlc queries, background jobs, auth flows, email templates, storage integration, SSE endpoints, and templ views.

### `/templui <component or page description>`

Generates Go templ templates using the [templui](https://www.templui.com/) component library — 41 pre-built components (buttons, forms, cards, dialogs, sidebars, charts, etc.) styled with Tailwind CSS and HTMX-ready.

## Usage

```
/forge-init my-saas-app
/forge-build user registration with email verification
/templui dashboard sidebar with navigation and user avatar
```

## License

Apache-2.0
