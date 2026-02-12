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
| `c.SessionGet(key)` | Get typed session value |
| `c.SessionSet(key, value)` | Set typed session value |

---

## Login Handler

```go
type loginRequest struct {
    Email    string `json:"email"    form:"email"    validate:"required;email" sanitize:"trim;lower"`
    Password string `json:"password" form:"password" validate:"required"`
}

func (h *AuthHandler) login(c forge.Context) error {
    var req loginRequest
    if err := c.Bind(&req); err != nil {
        return err
    }

    user, err := h.repo.GetUserByEmail(c, req.Email)
    if err != nil {
        return forge.ErrUnauthorized("invalid credentials")
    }

    if !verifyPassword(user.PasswordHash, req.Password) {
        return forge.ErrUnauthorized("invalid credentials")
    }

    c.AuthenticateSession(user.ID)

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
    if err := c.Bind(&req); err != nil {
        return err
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
    c.AuthenticateSession(user.ID)

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
        return c.Error(err)
    }

    c.AuthenticateSession(user.ID)
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
        r.GET("/oauth/:provider", h.oauthRedirect)
        r.GET("/oauth/:provider/callback", h.oauthCallback)
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
c.SessionSet("onboarding_step", 3)
c.SessionSet("selected_org", orgID)

// Retrieve typed value
step := c.SessionGet[int]("onboarding_step")
orgID := c.SessionGet[string]("selected_org")
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
