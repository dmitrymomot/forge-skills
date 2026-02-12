# One-Time Background Tasks

Loaded when the feature involves async jobs, background processing, or queued work.

---

## Task Definition

A task is a struct with `Name()` and `Handle()` methods:

```go
package task

import (
    "context"
    "log/slog"
)

// SendWelcomeEmailArgs defines the payload for the welcome email task.
type SendWelcomeEmailArgs struct {
    UserID string `json:"user_id"`
    Email  string `json:"email"`
    Name   string `json:"name"`
}

// SendWelcomeEmailWorker processes welcome email tasks.
type SendWelcomeEmailWorker struct {
    mailer *mailer.Mailer
    repo   *repository.Queries
}

func NewSendWelcomeEmailWorker(m *mailer.Mailer, repo *repository.Queries) *SendWelcomeEmailWorker {
    return &SendWelcomeEmailWorker{mailer: m, repo: repo}
}

func (w *SendWelcomeEmailWorker) Name() string {
    return "send_welcome_email"
}

func (w *SendWelcomeEmailWorker) Handle(ctx context.Context, args SendWelcomeEmailArgs) error {
    return w.mailer.Send(ctx, mailer.SendParams{
        To:       args.Email,
        Template: "welcome",
        Data:     map[string]any{"Name": args.Name},
    })
}
```

File location: `internal/task/<name>.go`

---

## Args Struct

- Use JSON tags for serialization
- Keep payloads small — IDs and minimal data, not full objects
- Args are stored in the job queue database table

```go
type ProcessImageArgs struct {
    FileKey string `json:"file_key"`
    UserID  string `json:"user_id"`
    Width   int    `json:"width"`
    Height  int    `json:"height"`
}
```

---

## Worker Registration

Register with `job.WithTask` in the job options:

```go
// In cmd/main.go
forge.WithJobs(jobPool, job.Config{},
    job.WithTask[task.SendWelcomeEmailArgs](
        task.NewSendWelcomeEmailWorker(mailerClient, repo),
    ),
    job.WithTask[task.ProcessImageArgs](
        task.NewProcessImageWorker(storageClient),
    ),
)
```

Import: `"myapp/internal/task"`

---

## Enqueue from Handler

```go
func (h *UserHandler) register(c forge.Context) error {
    // ... create user ...

    err := c.Enqueue("send_welcome_email", task.SendWelcomeEmailArgs{
        UserID: user.ID,
        Email:  user.Email,
        Name:   user.Name,
    })
    if err != nil {
        c.LogError("failed to enqueue welcome email", "error", err)
        // Non-fatal — user was created successfully
    }

    return c.JSON(http.StatusCreated, user)
}
```

### Enqueue within Transaction

```go
err := c.EnqueueTx(tx, "send_welcome_email", task.SendWelcomeEmailArgs{
    UserID: user.ID,
})
```

Job is only enqueued if the transaction commits.

---

## Enqueue Options

```go
c.Enqueue("task_name", payload,
    job.WithScheduledIn(5 * time.Minute),        // delay execution
    job.WithScheduledAt(time.Now().Add(1*time.Hour)), // specific time
    job.WithQueue("critical"),                    // custom queue
    job.WithMaxAttempts(5),                       // retry up to 5 times
    job.WithUniqueFor(10 * time.Minute),          // dedup window
    job.WithUniqueKey("user:" + userID),          // custom dedup key
    job.WithPriority(1),                          // lower number = higher priority
    job.WithTags("email", "onboarding"),          // metadata tags
)
```

---

## Retry Behavior

- Default: 25 attempts with exponential backoff (River default)
- Return error from `Handle()` to trigger retry
- Return `nil` to mark complete
- Use `job.WithMaxAttempts(n)` to customize
- After max attempts, job moves to dead letter queue

---

## Error Handling

```go
func (w *Worker) Handle(ctx context.Context, args Args) error {
    // Retryable error — return it
    result, err := w.service.Process(ctx, args.ID)
    if err != nil {
        return fmt.Errorf("processing %s: %w", args.ID, err)
    }

    // Non-retryable — log and return nil
    if result == nil {
        slog.Warn("item not found, skipping", "id", args.ID)
        return nil
    }

    return nil
}
```
