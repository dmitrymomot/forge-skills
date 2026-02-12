# docker-compose.yml — Docker Service Snippets

Generate `docker-compose.yml` at the project root by assembling the base structure with service snippets from each enabled subsystem.

## Base Structure

```yaml
services:
  # {{SERVICES}}

volumes:
  # {{VOLUMES}}
```

## Assembly Rules

1. Start with the base structure.
2. For each enabled subsystem that has a Docker Service section, insert its service definition under `services:`.
3. For each enabled subsystem that has a Docker Volume section, insert its volume definition under `volumes:`.
4. If no subsystems have Docker services (db, redis, storage) and mailer is not enabled, do NOT generate `docker-compose.yml`.
5. Replace `{{APP_NAME}}` in service configs with the actual app name (used in DB names, bucket names, etc.).

## Service Sources

Each subsystem's Docker service and volume definitions are found in their respective `data/subsystems/*.md` files under the "Docker Service" and "Docker Volume" sections:

- **db** → PostgreSQL service + `postgres_data` volume
- **redis** → Redis service + `redis_data` volume
- **storage** → MinIO service + MinIO init (bucket creation) + `minio_data` volume

- **mailer** → Mailpit service (no volume needed)

Other subsystems (sessions, jobs, htmx, oauth) do not have Docker services.

## Example Output (db + redis + storage + mailer)

```yaml
services:
  postgres:
    image: postgres:18-alpine
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: myapp
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      timeout: 5s
      retries: 5

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

  minio-init:
    image: minio/mc:latest
    depends_on:
      minio:
        condition: service_healthy
    restart: "no"
    entrypoint: >
      /bin/sh -c "
      mc alias set local http://minio:9000 minioadmin minioadmin;
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

volumes:
  postgres_data:
  redis_data:
  minio_data:
```

Note: The example above shows all services. In practice, only include the services for enabled subsystems. The `mailpit` service is included when `mailer` is enabled.

