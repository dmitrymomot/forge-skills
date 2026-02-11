# File Storage Subsystem (S3-compatible)

## Imports (config.go)

```go
"github.com/dmitrymomot/forge/pkg/storage"
```

## Imports (main.go)

```go
"github.com/dmitrymomot/forge/pkg/storage"
```

## Config Field

```go
Storage storage.Config `envPrefix:"STORAGE_"`
```

## Init Code

```go
s3, err := storage.NewS3(cfg.Storage)
if err != nil {
    log.Fatal("failed to init storage: ", err)
}
```

## Health Check

None.

## App Option

```go
forge.WithStorage(s3),
```

## Shutdown Hook

None.

## Env Vars (dev)

```env
STORAGE_BUCKET={{APP_NAME}}-files
STORAGE_ACCESS_KEY=minioadmin
STORAGE_SECRET_KEY=minioadmin
STORAGE_ENDPOINT=http://localhost:9000
STORAGE_REGION=us-east-1
STORAGE_PATH_STYLE=true
```

## Env Vars (example)

```env
STORAGE_BUCKET=your-bucket-name
STORAGE_ACCESS_KEY=<your-access-key>
STORAGE_SECRET_KEY=<your-secret-key>
STORAGE_ENDPOINT=https://s3.amazonaws.com
STORAGE_REGION=us-east-1
STORAGE_PATH_STYLE=false
```

## Docker Service

```yaml
minio:
  image: minio/minio:latest
  command: server /data --console-address ":9001"
  environment:
    MINIO_ROOT_USER: minioadmin
    MINIO_ROOT_PASSWORD: minioadmin
  ports:
    - "9000:9000"
    - "9001:9001"
  volumes:
    - minio_data:/data
  healthcheck:
    test: ["CMD", "mc", "ready", "local"]
    interval: 5s
    timeout: 5s
    retries: 5
```

## Docker Volume

```yaml
minio_data:
```

## Notes

- Storage is S3-compatible — works with AWS S3, MinIO, DigitalOcean Spaces, etc.
- MinIO is used for local development (docker-compose).
- `STORAGE_PATH_STYLE=true` is required for MinIO (path-style access).
- Available methods on `forge.Context`: `c.Upload()`, `c.FileURL()`.
