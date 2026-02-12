# Server-Sent Events (SSE)

Loaded when the feature involves real-time updates, live streaming, or push notifications.

---

## SSE Endpoint

```go
func (h *NotificationHandler) stream(c forge.Context) error {
    events := make(chan forge.SSEEvent, 10)

    go func() {
        defer close(events)
        ctx := c.Request().Context()

        for {
            select {
            case <-ctx.Done():
                return
            default:
                // Produce events (poll DB, listen to channel, etc.)
                notification, err := h.waitForNotification(ctx, c.UserID())
                if err != nil {
                    return
                }
                events <- forge.SSEJSON(notification)
            }
        }
    }()

    return c.SSE(events)
}
```

---

## Event Constructors

| Constructor | Description | Content-Type |
|---|---|---|
| `forge.SSEString(data)` | Plain text event | `text/plain` |
| `forge.SSEJSON(data)` | JSON-encoded event | `application/json` |
| `forge.SSETempl(component)` | HTML from templ component | `text/html` |
| `forge.SSEComment(text)` | Keepalive comment (`:text`) | — |
| `forge.SSERetry(millis)` | Set reconnection interval | — |

---

## Channel Pattern

```go
func (h *Handler) stream(c forge.Context) error {
    events := make(chan forge.SSEEvent, 10)

    go func() {
        defer close(events)
        ctx := c.Request().Context()

        ticker := time.NewTicker(5 * time.Second)
        defer ticker.Stop()

        for {
            select {
            case <-ctx.Done():
                return
            case <-ticker.C:
                // Poll for updates
                count, _ := h.repo.CountUnreadNotifications(ctx, c.UserID())
                events <- forge.SSEJSON(map[string]int64{"unread": count})
            }
        }
    }()

    return c.SSE(events)
}
```

---

## Context Cancellation

Always check `ctx.Done()` in the event producer goroutine:

```go
go func() {
    defer close(events)
    ctx := c.Request().Context()
    userID := c.UserID() // Extract before goroutine — session methods NOT goroutine-safe

    for {
        select {
        case <-ctx.Done():
            return
        case msg := <-subscription:
            events <- forge.SSEJSON(msg)
        }
    }
}()
```

**Important:** Extract session values (`c.UserID()`, `c.Role()`, etc.) BEFORE the goroutine.

---

## Keepalive

Enable app-wide SSE keepalive:

```go
// In cmd/main.go
forge.New(
    forge.WithSSEKeepAlive(30 * time.Second),
)
```

Sends `: keepalive` comment to prevent connection timeout.

---

## Client-Side: HTMX SSE Extension

```html
<div hx-ext="sse" sse-connect="/notifications/stream">
    <div sse-swap="message">
        <!-- Content replaced on each event -->
    </div>
</div>
```

Named events:

```html
<div hx-ext="sse" sse-connect="/events/stream">
    <div sse-swap="notification">Waiting for notifications...</div>
    <div sse-swap="progress">0%</div>
</div>
```

Include the SSE extension:

```html
<script src="https://unpkg.com/htmx-ext-sse@2"></script>
```

---

## Client-Side: EventSource API

For non-HTMX clients:

```javascript
const source = new EventSource('/notifications/stream');

source.onmessage = (event) => {
    const data = JSON.parse(event.data);
    // Handle event
};

source.onerror = () => {
    // Will auto-reconnect
};
```

---

## Common SSE Patterns

### Live Notifications

```go
func (h *NotificationHandler) Routes(r forge.Router) {
    r.Route("/notifications", func(r forge.Router) {
        r.Use(middlewares.RequireAuthenticated())
        r.GET("/stream", h.stream)
    })
}
```

### Progress Updates

```go
go func() {
    defer close(events)
    for i := range 100 {
        select {
        case <-ctx.Done():
            return
        default:
            events <- forge.SSEJSON(map[string]int{"progress": i + 1})
            time.Sleep(100 * time.Millisecond)
        }
    }
}()
```

### Real-Time Dashboard

```go
go func() {
    defer close(events)
    ticker := time.NewTicker(10 * time.Second)
    defer ticker.Stop()

    for {
        select {
        case <-ctx.Done():
            return
        case <-ticker.C:
            stats, _ := h.repo.GetDashboardStats(ctx)
            events <- forge.SSETempl(view.DashboardStats(stats))
        }
    }
}()
```
