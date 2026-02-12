# Authentication & Authorization

Loaded when the feature involves login, registration, sessions, OAuth, or RBAC.

---

## Session Methods

| Method | Description |
|---|---|
| `c.AuthenticateSession(userID)` | Set user as authenticated |
| `c.DestroySession()` | Clear session (logout) |
| `c.IsAuthenticated()` | Check if session is active |
| `c.UserID()` | Get authenticated user ID |
| `c.Role()` | Get user's RBAC role |
| `c.Can(permission)` | Check if user has permission |
| `forge.SessionGet[T](c, key)` | Get typed session value |
| `forge.SessionSet(c, key, value)` | Set typed session value |

---

## Login Handler

```go
type loginRequest struct {
    Email    string `json:"email"    form:"email"    validate:"required;email" sanitize:"trim;lower"`
    Password string `json:"password" form:"password" validate:"required"`
}

func (h *AuthHandler) login(c forge.Context) error {
    var req loginRequest
    validationErrors, err := c.Bind(&req)
    if err != nil {
        return err
    }
    if validationErrors != nil {
        return c.JSON(http.StatusUnprocessableEntity, map[string]any{
            "error":  "validation failed",
            "fields": validationErrors,
        })
    }

    user, err := h.repo.GetUserByEmail(c, req.Email)
    if err != nil {
        return forge.ErrUnauthorized("invalid credentials")
    }

    if !verifyPassword(user.PasswordHash, req.Password) {
        return forge.ErrUnauthorized("invalid credentials")
    }

    if err := c.AuthenticateSession(user.ID); err != nil {
        c.LogError("failed to authenticate session", "error", err)
        return forge.ErrInternal("something went wrong")
    }

    return c.Redirect(http.StatusSeeOther, "/dashboard")
}
```

---

## Register Handler

```go
type registerRequest struct {
    Name     string `json:"name"     form:"name"     validate:"required;max:255" sanitize:"trim"`
    Email    string `json:"email"    form:"email"    validate:"required;email"   sanitize:"trim;lower"`
    Password string `json:"password" form:"password" validate:"required;min:8"`
}

func (h *AuthHandler) register(c forge.Context) error {
    var req registerRequest
    validationErrors, err := c.Bind(&req)
    if err != nil {
        return err
    }
    if validationErrors != nil {
        return c.JSON(http.StatusUnprocessableEntity, map[string]any{
            "error":  "validation failed",
            "fields": validationErrors,
        })
    }

    hash, err := hashPassword(req.Password)
    if err != nil {
        return forge.ErrInternal("failed to process registration")
    }

    err = h.repo.CreateUser(c, repository.CreateUserParams{
        ID:           id.New(),
        Name:         req.Name,
        Email:        req.Email,
        PasswordHash: hash,
    })
    if err != nil {
        return forge.ErrConflict("email already registered")
    }

    // Auto-login after registration
    user, _ := h.repo.GetUserByEmail(c, req.Email)
    if err := c.AuthenticateSession(user.ID); err != nil {
        c.LogError("failed to authenticate session", "error", err)
        return forge.ErrInternal("something went wrong")
    }

    return c.Redirect(http.StatusSeeOther, "/dashboard")
}
```

---

## Logout Handler

```go
func (h *AuthHandler) logout(c forge.Context) error {
    c.DestroySession()
    return c.Redirect(http.StatusSeeOther, "/")
}
```

---

## OAuth Callback Pattern

```go
func (h *AuthHandler) oauthCallback(c forge.Context) error {
    code := forge.QueryDefault[string](c, "code", "")
    if code == "" {
        return forge.ErrBadRequest("missing authorization code")
    }

    // Exchange code for token
    token, err := h.oauthProvider.Exchange(c, code)
    if err != nil {
        return forge.ErrUnauthorized("oauth exchange failed")
    }

    // Fetch user profile from provider
    profile, err := h.oauthProvider.UserInfo(c, token)
    if err != nil {
        return forge.ErrInternal("failed to fetch profile")
    }

    // Upsert user (create if new, update if existing)
    user, err := h.repo.UpsertOAuthUser(c, repository.UpsertOAuthUserParams{
        ID:        id.New(),
        Email:     profile.Email,
        Name:      profile.Name,
        AvatarURL: profile.AvatarURL,
        Provider:  profile.Provider,
    })
    if err != nil {
        c.LogError("failed to upsert oauth user", "error", err)
        return forge.ErrInternal("something went wrong")
    }

    if err := c.AuthenticateSession(user.ID); err != nil {
        c.LogError("failed to authenticate session", "error", err)
        return forge.ErrInternal("something went wrong")
    }
    return c.Redirect(http.StatusSeeOther, "/dashboard")
}
```

---

## Auth Routes Pattern

```go
func (h *AuthHandler) Routes(r forge.Router) {
    r.Route("/auth", func(r forge.Router) {
        r.GET("/login", h.loginPage)
        r.POST("/login", h.login)
        r.GET("/register", h.registerPage)
        r.POST("/register", h.register)
        r.POST("/logout", h.logout)

        // OAuth
        r.GET("/oauth/{provider}", h.oauthRedirect)
        r.GET("/oauth/{provider}/callback", h.oauthCallback)
    })
}
```

---

## Auth Middleware on Routes

```go
// Protect entire route group
r.Route("/dashboard", func(r forge.Router) {
    r.Use(middlewares.RequireAuthenticated())
    r.GET("/", h.dashboard)
    r.GET("/settings", h.settings)
})

// RBAC permission check
r.Route("/admin", func(r forge.Router) {
    r.Use(middlewares.RequireAuthenticated())
    r.Use(middlewares.RequirePermission("admin:access"))
    r.GET("/users", h.listUsers)
})

// Any of multiple permissions
r.Route("/reports", func(r forge.Router) {
    r.Use(middlewares.RequireAnyPermission("reports:view", "admin:access"))
    r.GET("/", h.listReports)
})
```

---

## RBAC Inline Checks

```go
func (h *Handler) update(c forge.Context) error {
    post, _ := h.repo.GetPostByID(c, postID)

    // Owner or admin can update
    if post.UserID != c.UserID() && !c.Can("posts:update_any") {
        return forge.ErrForbidden("not allowed")
    }

    // ...
}
```

---

## Typed Session Helpers

```go
// Store typed value in session
forge.SessionSet(c, "onboarding_step", 3)
forge.SessionSet(c, "selected_org", orgID)

// Retrieve typed value
step := forge.SessionGet[int](c, "onboarding_step")
orgID := forge.SessionGet[string](c, "selected_org")
```

---

## Password Hashing

Use `golang.org/x/crypto/bcrypt`:

```go
import "golang.org/x/crypto/bcrypt"

func hashPassword(password string) (string, error) {
    hash, err := bcrypt.GenerateFromPassword([]byte(password), bcrypt.DefaultCost)
    return string(hash), err
}

func verifyPassword(hash, password string) bool {
    return bcrypt.CompareHashAndPassword([]byte(hash), []byte(password)) == nil
}
```
