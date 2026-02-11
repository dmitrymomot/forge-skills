# Redis Subsystem

## Imports (config.go)

```go
forgeredis "github.com/dmitrymomot/forge/pkg/redis"
```

## Imports (main.go)

```go
forgeredis "github.com/dmitrymomot/forge/pkg/redis"
```

## Config Field

```go
Redis forgeredis.Config `envPrefix:"REDIS_"`
```

## Init Code

```go
redisClient, err := forgeredis.Open(ctx, cfg.Redis)
if err != nil {
    log.Fatal("failed to connect to redis: ", err)
}
```

## Health Check

```go
forge.HealthCheck("redis", forgeredis.Healthcheck(redisClient)),
```

## App Option

None — Redis client is passed to handlers and middleware as a dependency.

## Shutdown Hook

```go
forge.WithShutdownHook(forgeredis.Shutdown(redisClient)),
```

## Env Vars (dev)

```env
REDIS_URL=redis://localhost:6379/0
```

## Env Vars (example)

```env
REDIS_URL=redis://localhost:6379/0
```

## Docker Service

```yaml
redis:
  image: redis:8-alpine
  ports:
    - "6379:6379"
  volumes:
    - redis_data:/data
  healthcheck:
    test: ["CMD", "redis-cli", "ping"]
    interval: 5s
    timeout: 5s
    retries: 5
```

## Docker Volume

```yaml
redis_data:
```
