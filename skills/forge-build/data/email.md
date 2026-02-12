# Email Templates & Sending

Loaded when the feature involves email notifications, transactional emails, or email templates.

---

## Template Format

Email templates use Markdown with YAML frontmatter:

```markdown
---
Subject: "Welcome to {{.AppName}}, {{.Name}}!"
---

Hi {{.Name}},

Thanks for signing up! We're excited to have you on board.

Here are some things you can do to get started:

- Complete your profile
- Explore the dashboard
- Invite your team members

[!button|Get Started]({{.DashboardURL}})

If you have any questions, just reply to this email.

Best,
The {{.AppName}} Team
```

File location: `templates/emails/<name>.md`

---

## Template Variables

Use Go template syntax: `{{.VariableName}}`

Pass data as `map[string]any`:

```go
data := map[string]any{
    "Name":         user.Name,
    "AppName":      "MyApp",
    "DashboardURL": "https://app.example.com/dashboard",
    "Code":         verificationCode,
}
```

---

## Button Syntax

```markdown
[!button|Button Label]({{.URL}})
```

Renders as a styled CTA button in the email.

---

## Layout File

Base HTML layout at `templates/emails/layouts/base.html`:

```html
<!DOCTYPE html>
<html>
<head><meta charset="utf-8"></head>
<body>
    <div style="max-width: 600px; margin: 0 auto; padding: 20px;">
        {{.Content}}
    </div>
</body>
</html>
```

`{{.Content}}` is replaced with the rendered markdown template.

---

## Embed File

Create `templates/emails/embed.go` for embedding:

```go
package emails

import "embed"

//go:embed *.md layouts/*.html
var FS embed.FS
```

---

## Mailer Setup

In `cmd/main.go` or initialization:

```go
import (
    "myapp/templates/emails"
    "github.com/dmitrymomot/forge/pkg/mailer"
    "github.com/dmitrymomot/forge/pkg/mailer/smtp"
)

// Create renderer with embedded templates
renderer := mailer.NewRenderer(emails.FS, "layouts/base.html")

// Create SMTP sender
sender := smtp.New(cfg.SMTP)

// Create mailer
m := mailer.New(sender, renderer, cfg.Mailer)
```

---

## Sending from Handler

### Synchronous (simple, blocks request)

```go
func (h *AuthHandler) register(c forge.Context) error {
    // ... create user ...

    err := h.mailer.Send(c, mailer.SendParams{
        To:       user.Email,
        Template: "welcome",
        Data: map[string]any{
            "Name":         user.Name,
            "DashboardURL": h.baseURL + "/dashboard",
        },
    })
    if err != nil {
        c.LogError("failed to send welcome email", "error", err)
    }

    return c.JSON(http.StatusCreated, user)
}
```

### Async via Job (preferred for production)

```go
// Enqueue email task instead of sending directly
err := c.Enqueue("send_email", task.SendEmailArgs{
    To:       user.Email,
    Template: "welcome",
    Data: map[string]any{
        "Name": user.Name,
    },
})
```

See `data/task.md` for the worker definition pattern.

---

## Config

In `cmd/config.go`:

```go
type Config struct {
    Mailer mailer.Config `envPrefix:"MAILER_"`
    SMTP   smtp.Config   `envPrefix:"SMTP_"`
}
```

Env vars: `MAILER_FROM`, `MAILER_FROM_NAME`, `SMTP_HOST`, `SMTP_PORT`, `SMTP_USER`, `SMTP_PASSWORD`

---

## Common Templates

| Template | Trigger | Variables |
|---|---|---|
| `welcome.md` | User registration | `Name`, `DashboardURL` |
| `verify-email.md` | Email verification | `Name`, `VerifyURL`, `Code` |
| `reset-password.md` | Password reset | `Name`, `ResetURL` |
| `invite.md` | Team invitation | `InviterName`, `TeamName`, `InviteURL` |
| `notification.md` | Generic notification | `Title`, `Body`, `ActionURL`, `ActionLabel` |
