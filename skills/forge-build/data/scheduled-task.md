# Scheduled (Periodic/Cron) Tasks

Loaded when the feature involves recurring jobs, cron schedules, or periodic processing.

---

## Scheduled Task Definition

A scheduled task has `Name()`, `Schedule()`, and `Handle()` — no payload:

```go
package task

import (
    "context"
    "log/slog"
)

// CleanupExpiredTokensTask removes expired tokens on a schedule.
type CleanupExpiredTokensTask struct {
    repo *repository.Queries
}

func NewCleanupExpiredTokensTask(repo *repository.Queries) *CleanupExpiredTokensTask {
    return &CleanupExpiredTokensTask{repo: repo}
}

func (t *CleanupExpiredTokensTask) Name() string {
    return "cleanup_expired_tokens"
}

func (t *CleanupExpiredTokensTask) Schedule() string {
    return "@daily"
}

func (t *CleanupExpiredTokensTask) Handle(ctx context.Context) error {
    count, err := t.repo.DeleteExpiredTokens(ctx)
    if err != nil {
        return fmt.Errorf("cleanup expired tokens: %w", err)
    }
    slog.Info("cleaned up expired tokens", "count", count)
    return nil
}
```

File location: `internal/task/<name>.go` (same directory as one-time tasks)

---

## Schedule Expressions

| Expression | Meaning |
|---|---|
| `@yearly` | Once a year (Jan 1, midnight) |
| `@monthly` | First of every month (midnight) |
| `@weekly` | Every Sunday (midnight) |
| `@daily` | Every day (midnight) |
| `@hourly` | Every hour |
| `@every 5m` | Every 5 minutes |
| `@every 30s` | Every 30 seconds |
| `@every 2h` | Every 2 hours |
| `0 */2 * * *` | Full cron: every 2 hours |
| `30 9 * * 1-5` | Full cron: 9:30 AM weekdays |
| `0 0 1 * *` | Full cron: first of each month |

---

## Registration

Register with `forge.WithScheduledTask` in job options:

```go
// In cmd/main.go
forge.WithJobs(jobPool,
    forge.WithScheduledTask(
        task.NewCleanupExpiredTokensTask(repo),
    ),
    forge.WithScheduledTask(
        task.NewSendDigestEmailTask(mailerClient, repo),
    ),
)
```

---

## Dependencies via Constructor

Inject any dependency the task needs:

```go
type SendWeeklyDigestTask struct {
    mailer *mailer.Mailer
    repo   *repository.Queries
}

func NewSendWeeklyDigestTask(m *mailer.Mailer, repo *repository.Queries) *SendWeeklyDigestTask {
    return &SendWeeklyDigestTask{mailer: m, repo: repo}
}
```

---

## Common Use Cases

| Task | Schedule | Description |
|---|---|---|
| Cleanup expired tokens | `@daily` | Remove expired auth/reset tokens |
| Send digest emails | `@weekly` or `0 9 * * 1` | Weekly summary to users |
| Cache warming | `@every 5m` | Pre-compute expensive queries |
| Report generation | `@monthly` | Generate monthly reports |
| Stale data cleanup | `@daily` | Remove orphaned records |
| Health check pings | `@every 1m` | External service monitoring |
| Usage metering | `@hourly` | Aggregate usage metrics |
