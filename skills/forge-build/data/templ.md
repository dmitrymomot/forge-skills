# Templ View Rendering

Loaded when the feature involves HTML pages, view templates, or server-rendered content.

---

## Templ Basics

`.templ` files compile to Go code. Each template is a function returning `templ.Component`:

```go
// internal/view/posts.templ

package view

import "myapp/internal/repository"

templ PostsPage(posts []repository.Post) {
    @Layout("Posts") {
        <div class="container">
            <h1>Posts</h1>
            @PostsList(posts)
        </div>
    }
}

templ PostsList(posts []repository.Post) {
    <div id="posts-list">
        for _, post := range posts {
            @PostCard(post)
        }
    </div>
}

templ PostCard(post repository.Post) {
    <div class="card">
        <h2>{ post.Title }</h2>
        <p>{ post.Body }</p>
    </div>
}
```

---

## Rendering from Handler

### Full page render

```go
func (h *PostHandler) listPage(c forge.Context) error {
    posts, err := h.repo.ListPosts(c, params)
    if err != nil {
        return c.Error(err)
    }
    return c.Render(http.StatusOK, view.PostsPage(posts))
}
```

### Partial render (HTMX-aware)

```go
func (h *PostHandler) listPage(c forge.Context) error {
    posts, err := h.repo.ListPosts(c, params)
    if err != nil {
        return c.Error(err)
    }
    return c.RenderPartial(http.StatusOK,
        view.PostsPage(posts),    // full page for browser navigation
        view.PostsList(posts),    // partial for HTMX requests
    )
}
```

---

## Layout Pattern

Base layout with `children` slot:

```go
// internal/view/layout.templ

package view

templ Layout(title string) {
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8"/>
        <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
        <title>{ title }</title>
        <script src="https://unpkg.com/htmx.org@2"></script>
        <link rel="stylesheet" href="/static/styles.css"/>
    </head>
    <body>
        @Nav()
        <main>
            { children... }
        </main>
    </body>
    </html>
}
```

---

## Page Pattern

Pages wrap layout + content:

```go
templ DashboardPage(stats DashboardStats) {
    @Layout("Dashboard") {
        <div class="dashboard">
            <h1>Dashboard</h1>
            @StatsGrid(stats)
            @RecentActivity(stats.RecentItems)
        </div>
    }
}
```

---

## Partial Pattern

Fragments for HTMX swaps — no layout wrapper:

```go
templ PostsList(posts []repository.Post) {
    <div id="posts-list">
        for _, post := range posts {
            <div class="post-item">
                <h3>{ post.Title }</h3>
                <p>{ post.Body }</p>
            </div>
        }
        if len(posts) == 0 {
            <p>No posts yet.</p>
        }
    </div>
}
```

---

## Data Passing

Templ functions take typed args from handler — use repository types directly or view models:

### Direct repository types

```go
templ UserProfile(user repository.User) {
    <h1>{ user.Name }</h1>
    <p>{ user.Email }</p>
}
```

### View model (when repository types don't match view needs)

```go
// internal/view/models.go
type PostListItem struct {
    ID        string
    Title     string
    Summary   string
    Author    string
    CreatedAt string // pre-formatted
}

// In handler:
items := make([]view.PostListItem, len(posts))
for i, p := range posts {
    items[i] = view.PostListItem{
        ID:        p.ID,
        Title:     p.Title,
        Summary:   truncate(p.Body, 100),
        Author:    p.AuthorName,
        CreatedAt: p.CreatedAt.Format("Jan 2, 2006"),
    }
}
return c.Render(http.StatusOK, view.PostsPage(items))
```

---

## File Organization

```
internal/view/
    layout.templ        # base layout
    nav.templ           # navigation component
    models.go           # view models (if needed)
    posts.templ         # posts page + partials
    dashboard.templ     # dashboard page
    auth.templ          # login/register pages
```

One package per domain or one flat package — keep it simple.

---

## Compilation

After creating/modifying `.templ` files:

```bash
go tool templ generate
```

Then verify:

```bash
go build -o /dev/null ./...
```

---

## Handoff to `/templui`

`forge-build` creates templ stubs with plain HTML (no styling).
Use `/templui` afterward to replace plain elements with styled templui components.
