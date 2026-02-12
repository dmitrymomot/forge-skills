# HTTP Handlers

Loaded when the feature involves endpoints, routes, or API handlers.

---

## Handler Struct Pattern

Every handler is a struct with dependencies, a constructor, and a `Routes()` method:

```go
package handler

import (
    "net/http"

    "github.com/dmitrymomot/forge"
    "github.com/dmitrymomot/forge/pkg/id"
    "github.com/jackc/pgx/v5/pgxpool"

    "myapp/internal/repository"
)

type PostHandler struct {
    repo *repository.Queries
}

func NewPostHandler(pool *pgxpool.Pool) *PostHandler {
    return &PostHandler{repo: repository.New(pool)}
}

func (h *PostHandler) Routes(r forge.Router) {
    r.GET("/posts", h.list)
    r.GET("/posts/:id", h.show)
    r.POST("/posts", h.create)
    r.PUT("/posts/:id", h.update)
    r.DELETE("/posts/:id", h.remove)
}
```

File location: `internal/handler/<name>.go`

---

## Request Structs

Bind tags: `json` for API, `form` for HTML forms, `query` for query params.
Validation and sanitization use semicolon separators.

```go
type createPostRequest struct {
    Title string `json:"title" form:"title" validate:"required;max:255" sanitize:"trim"`
    Body  string `json:"body"  form:"body"  validate:"required"         sanitize:"trim"`
}

type updatePostRequest struct {
    Title string `json:"title" form:"title" validate:"required;max:255" sanitize:"trim"`
    Body  string `json:"body"  form:"body"  validate:"required"         sanitize:"trim"`
}

type listPostsRequest struct {
    Page  int `query:"page"  validate:"min:1"`
    Limit int `query:"limit" validate:"min:1;max:100"`
}
```

---

## CRUD Handler Methods

### List (paginated)

```go
func (h *PostHandler) list(c forge.Context) error {
    page := forge.QueryDefault[int](c, "page", 1)
    limit := forge.QueryDefault[int](c, "limit", 20)
    offset := (page - 1) * limit

    posts, err := h.repo.ListPosts(c, repository.ListPostsParams{
        Limit:  int32(limit),
        Offset: int32(offset),
    })
    if err != nil {
        return c.Error(err)
    }

    return c.JSON(http.StatusOK, posts)
}
```

### Show (single)

```go
func (h *PostHandler) show(c forge.Context) error {
    postID := forge.Param[string](c, "id")

    post, err := h.repo.GetPostByID(c, postID)
    if err != nil {
        return forge.ErrNotFound("post not found")
    }

    return c.JSON(http.StatusOK, post)
}
```

### Create

```go
func (h *PostHandler) create(c forge.Context) error {
    var req createPostRequest
    if err := c.Bind(&req); err != nil {
        return err
    }

    err := h.repo.CreatePost(c, repository.CreatePostParams{
        ID:    id.New(),
        Title: req.Title,
        Body:  req.Body,
    })
    if err != nil {
        return c.Error(err)
    }

    return c.JSON(http.StatusCreated, map[string]string{"status": "created"})
}
```

### Update

```go
func (h *PostHandler) update(c forge.Context) error {
    postID := forge.Param[string](c, "id")

    var req updatePostRequest
    if err := c.Bind(&req); err != nil {
        return err
    }

    err := h.repo.UpdatePost(c, repository.UpdatePostParams{
        ID:    postID,
        Title: req.Title,
        Body:  req.Body,
    })
    if err != nil {
        return c.Error(err)
    }

    return c.JSON(http.StatusOK, map[string]string{"status": "updated"})
}
```

### Delete

```go
func (h *PostHandler) remove(c forge.Context) error {
    postID := forge.Param[string](c, "id")

    err := h.repo.DeletePost(c, postID)
    if err != nil {
        return c.Error(err)
    }

    return c.NoContent(http.StatusNoContent)
}
```

---

## Error Response Patterns

```go
// Not found
return forge.ErrNotFound("post not found")

// Validation error (422)
return c.JSON(http.StatusUnprocessableEntity, map[string]any{
    "error":  "validation failed",
    "fields": validationErrors,
})

// Bad request
return forge.ErrBadRequest("invalid input")

// Internal error (log + generic message)
c.LogError("failed to create post", "error", err)
return forge.ErrInternal("something went wrong")
```

---

## Route Grouping

```go
func (h *PostHandler) Routes(r forge.Router) {
    r.Route("/posts", func(r forge.Router) {
        r.GET("/", h.list)      // GET /posts
        r.POST("/", h.create)   // POST /posts

        r.Route("/:id", func(r forge.Router) {
            r.GET("/", h.show)      // GET /posts/:id
            r.PUT("/", h.update)    // PUT /posts/:id
            r.DELETE("/", h.remove) // DELETE /posts/:id
        })
    })
}
```

---

## Middleware on Routes

```go
func (h *PostHandler) Routes(r forge.Router) {
    r.Route("/posts", func(r forge.Router) {
        r.GET("/", h.list) // public

        r.Route("/", func(r forge.Router) {
            r.Use(middlewares.RequireAuthenticated())
            r.POST("/", h.create)    // auth required
            r.PUT("/:id", h.update)  // auth required
            r.DELETE("/:id", h.remove)
        })
    })
}
```

---

## Registration in main.go

Add handler to the `forge.WithHandlers()` option in `cmd/main.go`:

```go
forge.WithHandlers(
    handler.NewPostHandler(pool),
    // ... other handlers
),
```

Import: `"myapp/internal/handler"`

---

## HTML Response (templ)

For handlers that render HTML instead of JSON:

```go
func (h *PostHandler) list(c forge.Context) error {
    posts, err := h.repo.ListPosts(c, params)
    if err != nil {
        return c.Error(err)
    }
    return c.RenderPartial(http.StatusOK,
        view.PostsPage(posts),    // full page for browser
        view.PostsList(posts),    // partial for HTMX
    )
}
```

See `data/templ.md` and `data/htmx.md` for details.
