# Email Subsystem (Resend)

## Imports (config.go)

```go
"github.com/dmitrymomot/forge/pkg/mailer"
"github.com/dmitrymomot/forge/pkg/mailer/resend"
```

## Imports (main.go)

```go
"{{MODULE_PATH}}/templates/emails"

"github.com/dmitrymomot/forge/pkg/mailer"
"github.com/dmitrymomot/forge/pkg/mailer/resend"
```

## Config Fields

```go
Mailer mailer.Config `envPrefix:"MAILER_"`
Resend resend.Config `envPrefix:"RESEND_"`
```

## Init Code

```go
emailSender := resend.NewSender(cfg.Resend)
emailRenderer := mailer.NewRenderer(emails.FS, mailer.RendererConfig{})
emailClient := mailer.New(emailSender, emailRenderer, cfg.Mailer)
_ = emailClient // pass to handlers that need email
```

## Health Check

None.

## App Option

None — the mailer is not a forge.Option. Pass it to handlers as a dependency.

## Shutdown Hook

None.

## Generated Files

### `templates/emails/embed.go`

```go
package emails

import "embed"

//go:embed *.md layouts
var FS embed.FS
```

### `templates/emails/welcome.md`

```markdown
---
Subject: Welcome to {{APP_NAME}} - Let's get started!
---

# Welcome to {{APP_NAME}}, {{.Name}}!

We're excited to have you on board. Please confirm your email address to activate your account.

[!button|Confirm Your Email]({{.ConfirmationURL}})

## Getting Started

Here's what you can do next:

1. **Complete your profile** — Add your details to personalize your experience
2. **Explore the dashboard** — Familiarize yourself with the interface
3. **Invite your team** — Collaborate with colleagues

## Need Help?

If you have any questions, just reply to this email — we're happy to help.

---

If you didn't create an account with {{APP_NAME}}, please ignore this email.
```

### `templates/emails/layouts/base.html`

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="color-scheme" content="light only">
    <meta name="supported-color-schemes" content="light">
    {{if .Metadata.Subject}}<title>{{.Metadata.Subject}}</title>{{end}}
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }
        body {
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
            line-height: 1.6;
            color: #3c3c43;
            background-color: transparent;
            margin: 0;
            padding: 20px 0;
        }
        .email-container {
            max-width: 600px;
            margin: 0 auto;
            background-color: #ffffff;
            border-radius: 8px;
        }
        .logo-section {
            padding: 32px 24px 36px 24px;
        }
        .email-content {
            padding: 0 24px 32px 24px;
        }
        h1 {
            color: #000000;
            font-size: 28px;
            font-weight: 700;
            margin: 0 0 20px 0;
            line-height: 1.3;
        }
        h2 {
            color: #000000;
            font-size: 20px;
            font-weight: 700;
            margin: 30px 0 15px 0;
        }
        p {
            margin: 0 0 20px 0;
            font-size: 16px;
            color: #3c3c43;
            line-height: 1.5;
        }
        a {
            color: #3c3c43;
            text-decoration: underline;
        }
        a:hover {
            text-decoration: underline;
        }
        ul, ol {
            margin: 0 0 20px 0;
            padding-left: 20px;
        }
        li {
            margin: 8px 0;
            color: #3c3c43;
            font-size: 16px;
        }
        strong {
            color: #000000;
            font-weight: 700;
        }
        .btn {
            display: inline-block;
            background-color: #000000;
            color: #ffffff !important;
            -webkit-text-fill-color: #ffffff;
            text-decoration: none !important;
            padding: 10px 28px;
            border-radius: 8px;
            font-weight: 500;
            font-size: 15px;
            margin: 10px 0 16px 0;
            color-scheme: light;
        }
        .btn:hover {
            background-color: #1a1a1a;
        }
        .footer {
            padding: 0 24px 32px 24px;
            text-align: left;
            font-size: 13px;
            color: #8e8e93;
        }
        .footer p {
            margin: 0;
            color: #8e8e93;
            font-size: 13px;
        }
        .footer a {
            color: #8e8e93;
            font-weight: 400;
        }
        img {
            max-width: 100%;
            height: auto;
            display: block;
            margin: 20px 0;
            border-radius: 8px;
        }
        hr {
            border: none;
            border-top: 1px solid #e5e5ea;
            margin: 30px 0 23px 0;
        }
    </style>
</head>
<body>
    <div class="email-container">
        <div class="logo-section">
            <!-- TODO: replace with your logo -->
            <img src="https://example.com/logo.png" alt="{{APP_NAME}}" style="width: 48px; margin: 0;">
        </div>
        <div class="email-content">
            {{.Content}}
        </div>
        <div class="footer">
            <p>&copy; 2025 {{APP_NAME}}. All rights reserved.</p>
        </div>
    </div>
</body>
</html>
```

## Env Vars (dev)

```env
MAILER_FROM_NAME={{APP_NAME}}
MAILER_FROM_EMAIL=noreply@localhost
RESEND_API_KEY=re_test_placeholder
```

## Env Vars (example)

```env
MAILER_FROM_NAME=Your App Name
MAILER_FROM_EMAIL=noreply@yourdomain.com
RESEND_API_KEY=<your-resend-api-key>
```

## Docker Service

None.

## Notes

- The mailer is NOT a forge.Option — it's created independently and passed to handlers.
- The email renderer is scaffolded with `mailer.NewRenderer(emails.FS, mailer.RendererConfig{})` using the embedded template FS.
- Creates `templates/emails/` and `templates/emails/layouts/` directories with working template files.
- `templates/emails/welcome.md` is an example template — customize or replace with your own templates.
- `templates/emails/layouts/base.html` is the HTML layout wrapping all emails — update the logo, footer, and styles to match your brand.
- Resend is the default email provider. The `mailer.Sender` interface allows swapping providers.
