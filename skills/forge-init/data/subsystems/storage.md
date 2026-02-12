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
STORAGE_ACCESS_KEY=admin
STORAGE_SECRET_KEY=admin123
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
storage:
  image: rustfs/rustfs:latest
  ports:
    - "9000:9000"
    - "9001:9001"
  environment:
    RUSTFS_ACCESS_KEY: admin
    RUSTFS_SECRET_KEY: admin123
  volumes:
    - storage_data:/data
  healthcheck:
    test: ["CMD-SHELL", "nc -z localhost 9000 || exit 1"]
    interval: 5s
    timeout: 5s
    retries: 5
    start_period: 5s

storage-bucket-init:
  image: minio/mc:latest
  depends_on:
    storage:
      condition: service_healthy
  restart: "no"
  entrypoint: >
    /bin/sh -c "
    mc alias set local http://storage:9000 admin admin123;
    mc mb --ignore-existing local/{{APP_NAME}}-files;
    "
```

## Docker Volume

```yaml
storage_data:
```

## Notes

- Storage is S3-compatible — works with AWS S3, RustFS, MinIO, DigitalOcean Spaces, etc.
- RustFS is used for local development (docker-compose).
- `STORAGE_PATH_STYLE=true` is required for RustFS (path-style access).
- Available methods on `forge.Context`: `c.Upload()`, `c.FileURL()`.
