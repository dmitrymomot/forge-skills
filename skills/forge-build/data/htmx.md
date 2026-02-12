# HTMX Request/Response Patterns

Loaded when the feature involves HTMX interactions, partial page updates, or dynamic UI.

---

## HTMX Detection

```go
if c.IsHTMX() {
    // Serve partial
} else {
    // Serve full page
}
```

Or use the built-in helper:

```go
return c.RenderPartial(http.StatusOK,
    view.FullPage(data),    // browser navigation
    view.Partial(data),     // HTMX request
)
```

---

## Common HTMX Attributes

Use these in templ templates:

| Attribute | Description |
|---|---|
| `hx-get="/url"` | GET request |
| `hx-post="/url"` | POST request |
| `hx-put="/url"` | PUT request |
| `hx-delete="/url"` | DELETE request |
| `hx-target="#id"` | Where to put response |
| `hx-swap="innerHTML"` | How to swap content |
| `hx-trigger="click"` | What triggers the request |
| `hx-push-url="true"` | Update browser URL |
| `hx-boost="true"` | Progressive enhancement |
| `hx-confirm="Sure?"` | Confirmation dialog |
| `hx-indicator="#spinner"` | Loading indicator |

---

## Swap Modes

| Mode | Behavior |
|---|---|
| `innerHTML` | Replace inner content (default) |
| `outerHTML` | Replace entire element |
| `beforeend` | Append inside element |
| `afterbegin` | Prepend inside element |
| `beforebegin` | Insert before element |
| `afterend` | Insert after element |
| `delete` | Remove the target |
| `none` | Don't swap (for side effects) |

---

## Trigger Patterns

```html
hx-trigger="click"
hx-trigger="submit"
hx-trigger="load"
hx-trigger="revealed"
hx-trigger="every 5s"
hx-trigger="keyup changed delay:300ms"
hx-trigger="click from:#other-element"
hx-trigger="intersect once"
```

---

## Response Headers from Handler

Set HTMX response headers for client-side behavior:

```go
// Redirect (HTMX-aware — uses HX-Redirect instead of 302)
return c.Redirect(http.StatusSeeOther, "/posts")

// Trigger client-side event
c.SetHeader("HX-Trigger", "itemCreated")

// Trigger with data
c.SetHeader("HX-Trigger", `{"itemCreated": {"id": "123"}}`)

// Change swap behavior
c.SetHeader("HX-Reswap", "outerHTML")

// Change target
c.SetHeader("HX-Retarget", "#other-element")

// Update URL without navigation
c.SetHeader("HX-Push-Url", "/posts/123")
```

---

## Active Search Pattern

Input that searches as you type:

```go
// Templ
templ SearchInput() {
    <input type="search"
        name="q"
        hx-get="/posts/search"
        hx-target="#search-results"
        hx-swap="innerHTML"
        hx-trigger="keyup changed delay:300ms"
        placeholder="Search posts..."/>
    <div id="search-results"></div>
}
```

Handler:

```go
func (h *PostHandler) search(c forge.Context) error {
    q := forge.QueryDefault[string](c, "q", "")
    posts, err := h.repo.SearchPosts(c, repository.SearchPostsParams{
        Query:  q,
        Limit:  20,
        Offset: 0,
    })
    if err != nil {
        return c.Error(err)
    }
    return c.Render(http.StatusOK, view.PostsList(posts))
}
```

---

## Inline Edit Pattern

Click to edit, submit to save:

```go
// Display mode
templ PostTitle(post repository.Post) {
    <span id={ "title-" + post.ID }
        hx-get={ "/posts/" + post.ID + "/edit-title" }
        hx-target={ "#title-" + post.ID }
        hx-swap="outerHTML"
        class="cursor-pointer">
        { post.Title }
    </span>
}

// Edit mode
templ PostTitleEdit(post repository.Post) {
    <form id={ "title-" + post.ID }
        hx-put={ "/posts/" + post.ID + "/title" }
        hx-target={ "#title-" + post.ID }
        hx-swap="outerHTML">
        <input type="text" name="title" value={ post.Title }/>
        <button type="submit">Save</button>
    </form>
}
```

---

## Infinite Scroll Pattern

```go
templ PostsList(posts []repository.Post, nextPage int) {
    for _, post := range posts {
        @PostCard(post)
    }
    if len(posts) > 0 {
        <div hx-get={ fmt.Sprintf("/posts?page=%d", nextPage) }
            hx-trigger="revealed"
            hx-swap="outerHTML"
            hx-target="this">
            Loading more...
        </div>
    }
}
```

---

## Delete with Confirmation

```go
templ DeleteButton(postID string) {
    <button
        hx-delete={ "/posts/" + postID }
        hx-target={ "#post-" + postID }
        hx-swap="outerHTML"
        hx-confirm="Are you sure you want to delete this post?">
        Delete
    </button>
}
```

Handler returns empty body or the updated list.

---

## Form Submission

```go
templ CreatePostForm() {
    <form hx-post="/posts"
        hx-target="#posts-list"
        hx-swap="beforeend"
        hx-on::after-request="this.reset()">
        <input type="text" name="title" required/>
        <textarea name="body" required></textarea>
        <button type="submit">Create</button>
    </form>
}
```
