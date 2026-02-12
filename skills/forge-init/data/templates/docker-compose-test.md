# docker-compose.test.yml — Ephemeral Test Infrastructure

Generate `docker-compose.test.yml` at the project root by assembling the base structure with service snippets from each enabled subsystem. This file uses **tmpfs** mounts instead of named volumes for fast, ephemeral test infrastructure.

## Base Structure

```yaml
services:
  # {{SERVICES}}
```

No `volumes:` top-level section is needed since everything uses tmpfs.

## Assembly Rules

1. Start with the base structure.
2. For each enabled subsystem that has a Docker service, insert its service definition under `services:`.
3. If no subsystems have Docker services (db, redis, storage) and mailer is not enabled, do NOT generate `docker-compose.test.yml`.
4. Replace `{{APP_NAME}}` in service configs with the actual app name (used in DB names, bucket names, etc.).

## Service Snippets

### db (PostgreSQL) — when `db` is enabled

```yaml
  postgres:
    image: postgres:18-alpine
    ports:
      - "5432:5432"
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: {{APP_NAME}}
    tmpfs:
      - /var/lib/postgresql:mode=1777
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      timeout: 5s
      retries: 5
```

### redis — when `redis` is enabled

```yaml
  redis:
    image: redis:8-alpine
    ports:
      - "6379:6379"
    tmpfs:
      - /data:mode=1777
    healthcheck:
      test: ["CMD-SHELL", "redis-cli ping | grep PONG"]
      interval: 5s
      timeout: 5s
      retries: 5
```

### storage (RustFS) — when `storage` is enabled

```yaml
  storage:
    image: rustfs/rustfs:latest
    ports:
      - "9000:9000"
      - "9001:9001"
    environment:
      RUSTFS_ACCESS_KEY: admin
      RUSTFS_SECRET_KEY: admin123
    tmpfs:
      - /data:mode=1777
      - /logs:mode=0755
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

### mailer (Mailpit) — when `mailer` is enabled

```yaml
  mailpit:
    image: axllent/mailpit:latest
    ports:
      - "1025:1025"
      - "8025:8025"
    healthcheck:
      test: ["CMD-SHELL", "nc -z localhost 1025"]
      interval: 5s
      timeout: 5s
      retries: 5
```

## Example Output (db + redis + storage + mailer)

```yaml
services:
  postgres:
    image: postgres:18-alpine
    ports:
      - "5432:5432"
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: myapp
    tmpfs:
      - /var/lib/postgresql:mode=1777
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      timeout: 5s
      retries: 5

  redis:
    image: redis:8-alpine
    ports:
      - "6379:6379"
    tmpfs:
      - /data:mode=1777
    healthcheck:
      test: ["CMD-SHELL", "redis-cli ping | grep PONG"]
      interval: 5s
      timeout: 5s
      retries: 5

  storage:
    image: rustfs/rustfs:latest
    ports:
      - "9000:9000"
      - "9001:9001"
    environment:
      RUSTFS_ACCESS_KEY: admin
      RUSTFS_SECRET_KEY: admin123
    tmpfs:
      - /data:mode=1777
      - /logs:mode=0755
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
      mc mb --ignore-existing local/myapp-files;
      "

  mailpit:
    image: axllent/mailpit:latest
    ports:
      - "1025:1025"
      - "8025:8025"
    healthcheck:
      test: ["CMD-SHELL", "nc -z localhost 1025"]
      interval: 5s
      timeout: 5s
      retries: 5
```
