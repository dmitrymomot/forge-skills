# OAuth Subsystem (Google/GitHub)

## Imports (config.go)

```go
"github.com/dmitrymomot/forge/pkg/oauth"
```

## Imports (main.go)

```go
"github.com/dmitrymomot/forge/pkg/oauth"
```

## Config Fields

```go
GoogleOAuth oauth.GoogleConfig `envPrefix:"GOOGLE_OAUTH_"`
GitHubOAuth oauth.GitHubConfig `envPrefix:"GITHUB_OAUTH_"`
```

## Init Code

```go
googleOAuth, err := oauth.NewGoogleProvider(cfg.GoogleOAuth)
if err != nil {
    log.Fatal("failed to init google oauth: ", err)
}
githubOAuth, err := oauth.NewGitHubProvider(cfg.GitHubOAuth)
if err != nil {
    log.Fatal("failed to init github oauth: ", err)
}
_, _ = googleOAuth, githubOAuth // pass to handlers that need OAuth
```

## Health Check

None.

## App Option

None — OAuth providers are not forge.Options. Pass them to handlers as dependencies.

## Shutdown Hook

None.

## Env Vars (dev)

```env
GOOGLE_OAUTH_CLIENT_ID=your-google-client-id
GOOGLE_OAUTH_CLIENT_SECRET=your-google-client-secret
GOOGLE_OAUTH_REDIRECT_URL=http://localhost:8080/auth/google/callback
GITHUB_OAUTH_CLIENT_ID=your-github-client-id
GITHUB_OAUTH_CLIENT_SECRET=your-github-client-secret
GITHUB_OAUTH_REDIRECT_URL=http://localhost:8080/auth/github/callback
```

## Env Vars (example)

```env
GOOGLE_OAUTH_CLIENT_ID=<your-google-client-id>
GOOGLE_OAUTH_CLIENT_SECRET=<your-google-client-secret>
GOOGLE_OAUTH_REDIRECT_URL=http://localhost:8080/auth/google/callback
GITHUB_OAUTH_CLIENT_ID=<your-github-client-id>
GITHUB_OAUTH_CLIENT_SECRET=<your-github-client-secret>
GITHUB_OAUTH_REDIRECT_URL=http://localhost:8080/auth/github/callback
```

## Docker Service

None.

## Notes

- OAuth providers are NOT forge.Options — they are created independently and passed to handlers.
- Sessions are strongly recommended for OAuth state management (CSRF protection).
- If sessions are not enabled, the skill should suggest adding them.
- Redirect URLs use `localhost:8080` for development; update for production.
- Both Google and GitHub providers are included by default. Remove unused ones.
