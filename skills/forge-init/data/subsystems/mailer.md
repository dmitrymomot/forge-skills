# Email Subsystem (Resend)

## Imports (config.go)

```go
"github.com/dmitrymomot/forge/pkg/mailer"
"github.com/dmitrymomot/forge/pkg/mailer/resend"
```

## Imports (main.go)

```go
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
// TODO: Provide your own template renderer implementation.
// The renderer must implement the mailer.Renderer interface.
var emailRenderer mailer.Renderer // replace with your implementation
emailClient := mailer.New(emailSender, emailRenderer, cfg.Mailer)
_ = emailClient // pass to handlers that need email
```

## Health Check

None.

## App Option

None — the mailer is not a forge.Option. Pass it to handlers as a dependency.

## Shutdown Hook

None.

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
- The email renderer is intentionally a TODO — user implements based on their template engine.
- Creates `templates/emails/` directory for email templates.
- Resend is the default email provider. The `mailer.Sender` interface allows swapping providers.
