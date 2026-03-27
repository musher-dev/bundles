---
name: composing-docker-development
version: 1.0.0
user-invocable: false
description: >-
  Design Docker and Docker Compose configurations for local development
  environments including multi-service stacks, hot-reload volume mounts, database
  and cache services, networking, health checks, development-specific overrides,
  Dockerfile multi-stage patterns, and .dockerignore optimization. Use when creating
  or reviewing docker-compose.yml for local dev workflows, designing multi-stage
  Dockerfiles, or troubleshooting container networking.
  Triggered by: docker compose, docker-compose, Dockerfile, docker development,
  multi-stage, docker volume, docker network, container development, compose yaml,
  dockerignore, docker health check, docker dev.
allowed-tools:
  - Read
  - Grep
  - Glob
  - Edit
  - Write
---

# Docker Development Composition

Patterns for designing Docker and Docker Compose configurations optimized for local development — fast rebuilds, hot-reload, infrastructure services, and production parity.

## Purpose

Docker Compose turns "install PostgreSQL, Redis, and Elasticsearch on your laptop" into `docker compose up`. But a development Docker setup has different requirements than production: it needs fast rebuilds, volume-mounted source code for hot-reload, accessible ports for debugging, and sensible defaults that work without configuration. This skill covers the patterns that make Docker development environments fast, reliable, and maintainable.

Use this skill when creating Docker Compose configurations for local development, designing multi-stage Dockerfiles that serve both dev and production, or troubleshooting service connectivity and performance in containerized environments.

---

## Docker Compose for Development

### Standard Structure

```yaml
# docker-compose.yml (or compose.yaml)
services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
      target: development
    volumes:
      - .:/app:cached
      - node_modules:/app/node_modules
    ports:
      - "8000:8000"
    env_file: .env
    environment:
      DATABASE_URL: postgresql://dev:dev@db:5432/app_dev
      REDIS_URL: redis://redis:6379/0
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_healthy
    command: uvicorn app.main:app --reload --host 0.0.0.0 --port 8000

  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: dev
      POSTGRES_PASSWORD: dev
      POSTGRES_DB: app_dev
    ports:
      - "5432:5432"
    volumes:
      - pgdata:/var/lib/postgresql/data
      - ./scripts/init-db.sql:/docker-entrypoint-initdb.d/init.sql:ro
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U dev -d app_dev"]
      interval: 5s
      timeout: 3s
      retries: 5
      start_period: 10s

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redisdata:/data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
      timeout: 3s
      retries: 5

volumes:
  pgdata:
  redisdata:
  node_modules:
```

---

## Multi-Stage Dockerfile

### Development + Production Pattern

```dockerfile
# syntax=docker/dockerfile:1

# ============================================================
# Base: shared dependencies
# ============================================================
FROM python:3.12-slim AS base

WORKDIR /app
ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1

# System dependencies needed by both dev and prod
RUN apt-get update && apt-get install -y --no-install-recommends \
    libpq-dev \
    && rm -rf /var/lib/apt/lists/*

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# ============================================================
# Development: hot-reload, debug tools
# ============================================================
FROM base AS development

# Dev-only dependencies
COPY requirements-dev.txt .
RUN pip install --no-cache-dir -r requirements-dev.txt

# Source code mounted as volume — not COPY'd
# CMD provided by docker-compose

# ============================================================
# Production: optimized, minimal
# ============================================================
FROM base AS production

COPY . .

RUN adduser --disabled-password --no-create-home appuser
USER appuser

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### Stage Selection

In `docker-compose.yml`:
```yaml
build:
  target: development  # Uses the dev stage
```

In CI/production:
```bash
docker build --target production -t myapp:latest .
```

---

## Volume Mount Strategy

### Mount Types by Use Case

| Content | Mount Type | Syntax | Rationale |
|---|---|---|---|
| Source code | Bind mount (cached) | `.:/app:cached` | Hot-reload from host edits |
| Dependencies | Named volume | `node_modules:/app/node_modules` | Avoid host/container mismatch, faster I/O |
| Database data | Named volume | `pgdata:/var/lib/postgresql/data` | Persist across container restarts |
| Build cache | Named volume | `buildcache:/app/.cache` | Faster rebuilds |
| Temp files | tmpfs | `tmpfs: /tmp` | Fast, no persistence needed |

### The node_modules Problem

Without a named volume:
```yaml
volumes:
  - .:/app  # Host node_modules overwrites container's
```

The host `node_modules/` (or absence thereof) overwrites the container's installed dependencies. Solution:

```yaml
volumes:
  - .:/app:cached
  - node_modules:/app/node_modules  # Named volume takes precedence
```

This pattern applies to any language's dependency directory: `vendor/`, `__pycache__/`, `.venv/`, etc.

---

## Health Checks

Every infrastructure service must have a health check. Without one, `depends_on` only waits for the container to start, not for the service to be ready.

### Common Health Checks

| Service | Health Check |
|---|---|
| PostgreSQL | `["CMD-SHELL", "pg_isready -U ${POSTGRES_USER} -d ${POSTGRES_DB}"]` |
| MySQL | `["CMD", "mysqladmin", "ping", "-h", "localhost"]` |
| Redis | `["CMD", "redis-cli", "ping"]` |
| Elasticsearch | `["CMD-SHELL", "curl -f http://localhost:9200/_cluster/health || exit 1"]` |
| RabbitMQ | `["CMD", "rabbitmq-diagnostics", "check_running"]` |
| MongoDB | `["CMD", "mongosh", "--eval", "db.adminCommand('ping')"]` |

### Health Check Timing

```yaml
healthcheck:
  test: ["CMD-SHELL", "pg_isready -U dev"]
  interval: 5s      # Time between checks
  timeout: 3s       # Max time for a single check
  retries: 5        # Failures before unhealthy
  start_period: 10s # Grace period for startup
```

**Rule:** Always specify `start_period` for services that take time to initialize (databases, Elasticsearch). Without it, the health check may fail and restart the container during normal startup.

---

## Networking

### Service Discovery

Services reference each other by service name:

```yaml
environment:
  DATABASE_URL: postgresql://dev:dev@db:5432/app_dev  # "db" is the service name
  REDIS_URL: redis://redis:6379/0                     # "redis" is the service name
```

**Common mistake:** Using `localhost` inside a container. `localhost` refers to the container itself, not the host or other containers.

### Port Mapping

```yaml
ports:
  - "5432:5432"   # host:container — accessible from host
  - "6379:6379"
```

**Rules:**
- Expose infrastructure ports for debugging tools (pgAdmin, Redis Commander)
- Use the same port number on host and container to avoid confusion
- If port conflicts occur, change the host port: `"5433:5432"`

### Host Networking (When Needed)

```yaml
# Access host services from container (e.g., host-installed database)
extra_hosts:
  - "host.docker.internal:host-gateway"
```

---

## Override Files

### Development Override Pattern

```yaml
# docker-compose.yml — base configuration
services:
  app:
    image: myapp:latest

# docker-compose.override.yml — auto-loaded for development
services:
  app:
    build:
      context: .
      target: development
    volumes:
      - .:/app:cached
    command: uvicorn app.main:app --reload --host 0.0.0.0
```

Docker Compose automatically merges `docker-compose.override.yml` on top of `docker-compose.yml`.

### Environment-Specific Compose Files

```bash
# Development (auto-merged)
docker compose up

# Testing
docker compose -f docker-compose.yml -f docker-compose.test.yml up

# Production
docker compose -f docker-compose.yml -f docker-compose.prod.yml up
```

---

## .dockerignore

```dockerignore
# Version control
.git
.gitignore

# Dependencies (installed in container)
node_modules
.venv
__pycache__

# Build artifacts
dist
build
*.egg-info

# IDE and editor
.vscode
.idea
*.swp
*.swo

# Environment (secrets)
.env
.env.local

# Docker (prevent recursive context)
docker-compose*.yml
Dockerfile*

# Documentation
*.md
LICENSE

# Tests (not needed in production image)
tests/
test/
```

**Rule:** Keep `.dockerignore` in sync with what the production image needs. A smaller build context means faster builds.

---

## Anti-Patterns

### 1. No Health Checks

```yaml
depends_on:
  - db  # Only waits for container start, not service readiness
```

Application crashes on startup because PostgreSQL isn't accepting connections yet. Always use `condition: service_healthy`.

### 2. Root User in Development Container

```dockerfile
# No USER instruction — runs as root
```

Files created inside the container are owned by root on the host, causing permission issues. Add a non-root user or use `user:` in compose.

### 3. Copying Source Code in Development Dockerfile

```dockerfile
FROM python:3.12
COPY . .  # In development, use volume mounts instead
CMD ["uvicorn", "app:main", "--reload"]
```

Requires a rebuild for every code change. Volume mount source code in development; only COPY in the production stage.

### 4. Missing Named Volumes for Dependencies

```yaml
volumes:
  - .:/app  # node_modules on host overwrites container's
```

Container dependencies conflict with (or are missing from) the host. Use named volumes for dependency directories.

### 5. Hardcoded Credentials in docker-compose.yml

```yaml
environment:
  POSTGRES_PASSWORD: super_secret_production_password
```

Compose files are committed. Use `env_file` or `${VARIABLE}` interpolation with `.env`.

### 6. Missing .dockerignore

The entire repository (including `.git/`, `node_modules/`, and `.env`) is sent as build context. Builds are slow and may leak secrets into the image.

---

## Checklist

Before committing Docker development configuration:

- [ ] Multi-stage Dockerfile with `development` and `production` targets
- [ ] Source code bind-mounted with `:cached` flag
- [ ] Named volumes for dependency directories (node_modules, .venv)
- [ ] Named volumes for database/cache persistence
- [ ] Health checks on all infrastructure services
- [ ] `depends_on` uses `condition: service_healthy`
- [ ] `start_period` set for slow-starting services
- [ ] Services reference each other by name, not `localhost`
- [ ] `.dockerignore` excludes .git, node_modules, .env, tests
- [ ] No hardcoded secrets in compose files
- [ ] Non-root user in Dockerfile
- [ ] Ports exposed for local debugging tools

---

## Related Skills and Agents

- **`configuring-devcontainers` skill**: Devcontainers can use Docker Compose for multi-service environments — this skill provides the Docker patterns that devcontainers reference
- **`managing-environment-variables` skill**: Docker Compose `env_file` and `environment` settings interact with `.env` management
- **`aligning-ci-environments` skill**: CI may reuse the same Docker images or Compose files — ensures parity
- **`devcontainer_specialist` agent**: Deep expertise in Docker + devcontainer integration
- **`environment_scaffolder` agent**: Generates Docker development configuration as part of complete environment setup
