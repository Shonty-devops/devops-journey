# Day 08 — Docker: build arguments (ARG)

**Platform:** KillerCoda  
**Date:** April 2026  
**Topic:** Docker best practices — dynamic builds using ARG

---

## What is a build argument?

`ARG` is a Dockerfile instruction that defines 
a variable passed at BUILD TIME using `--build-arg` 
flag. Unlike `ENV`, build arguments are NOT 
available when the container is running — 
they only exist during the image build process.

---

## ARG vs ENV — key difference

| | ARG | ENV |
|--|-----|-----|
| Available at | Build time only | Build time + Runtime |
| Defined in | Dockerfile with `ARG` | Dockerfile with `ENV` |
| Passed via | `--build-arg` flag | `-e` flag or `docker-compose` |
| Visible in image | No | Yes |
| Use case | Version numbers, build config | App config, credentials |

---

## The Dockerfile — before and after

```dockerfile
# BEFORE — hardcoded Go version
FROM golang:1.21 AS builder
WORKDIR /app
COPY . .
RUN go build -o server .

FROM scratch
COPY --from=builder /app/server /server
ENTRYPOINT ["/server"]

# AFTER — dynamic Go version using ARG
ARG GO_VERSION=1.21
FROM golang:${GO_VERSION} AS builder
WORKDIR /app
COPY . .
RUN go build -o server .

FROM scratch
COPY --from=builder /app/server /server
ENTRYPOINT ["/server"]
```

---

## Full solution — step by step

```bash
# Step 1 — edit the Dockerfile
vim /root/Dockerfile
# Add ARG GO_VERSION=1.21 BEFORE the first FROM
# Change FROM golang:1.21 to FROM golang:${GO_VERSION}
# Save: ESC → :wq

# Step 2 — build image with specific Go version
docker build --build-arg GO_VERSION=1.22 -t myapp /root
# --build-arg GO_VERSION=1.22 overrides the default 1.21
# Docker pulls golang:1.22 as the base image

# Step 3 — verify the image was built
docker image ls
# Output:
# REPOSITORY   TAG      IMAGE ID       SIZE
# myapp        latest   abc123def456   10MB
```

---

## How ARG works visually

```
Dockerfile:
ARG GO_VERSION=1.21          ← default value
FROM golang:${GO_VERSION}    ← uses the ARG value

Build without --build-arg:
docker build -t myapp .
→ uses default: golang:1.21

Build with --build-arg:
docker build --build-arg GO_VERSION=1.22 -t myapp .
→ overrides default: golang:1.22
```

---

## Important rule — ARG before FROM

```dockerfile
# ARG declared BEFORE FROM affects only the FROM line
ARG GO_VERSION=1.21
FROM golang:${GO_VERSION} AS builder

# To use ARG AFTER FROM, you must re-declare it
ARG GO_VERSION          # re-declare (no default needed)
RUN echo "Building with Go ${GO_VERSION}"
```

This is a common gotcha — ARG scope resets 
after each FROM statement.

---

## Multiple build arguments

```dockerfile
ARG GO_VERSION=1.21
ARG APP_VERSION=1.0.0
ARG BUILD_ENV=production

FROM golang:${GO_VERSION} AS builder
ARG APP_VERSION
ARG BUILD_ENV

WORKDIR /app
COPY . .
RUN go build -ldflags="-X main.version=${APP_VERSION}" -o server .
```

```bash
# Build with multiple args
docker build \
  --build-arg GO_VERSION=1.22 \
  --build-arg APP_VERSION=2.1.0 \
  --build-arg BUILD_ENV=staging \
  -t myapp:2.1.0 .
```

---

## Real DevOps CI/CD example

```bash
# In a Jenkins or GitHub Actions pipeline:
APP_VERSION=$(git describe --tags)
GO_VERSION="1.22"

docker build \
  --build-arg GO_VERSION=${GO_VERSION} \
  --build-arg APP_VERSION=${APP_VERSION} \
  -t myapp:${APP_VERSION} \
  -t myapp:latest \
  .

# This lets you:
# - Control exact Go version without editing Dockerfile
# - Inject version number from git tags automatically
# - Same Dockerfile works for all versions
```

---

## Security warning — do NOT use ARG for secrets

```dockerfile
# NEVER do this — build args can be exposed
# via docker history command
ARG DB_PASSWORD=secret123     # WRONG
ARG API_KEY=abc123            # WRONG

# Use Docker secrets or environment variables
# for sensitive data at runtime instead
```

```bash
# Anyone can see build args with:
docker history myapp
# ARG values appear in the layer history
```

---

## Key takeaways for DevOps

- `ARG` makes Dockerfiles reusable across 
  versions — no need to edit the file for 
  every version change
- Always set a sensible default value for ARG — 
  `ARG GO_VERSION=1.21` so the build works 
  without `--build-arg` if needed
- In CI/CD pipelines, `--build-arg` is used 
  to inject version numbers, environment names, 
  and build metadata automatically
- Never pass secrets via `ARG` — they are 
  visible in `docker history`
- `ARG` before `FROM` only controls the `FROM` 
  line — re-declare inside the stage to use it 
  in `RUN` commands

---

## New commands learned today

| Command | Purpose |
|---------|---------|
| `ARG <name>=<default>` | Declare build argument with default value |
| `${ARG_NAME}` | Reference ARG value in Dockerfile |
| `docker build --build-arg <name>=<value>` | Override ARG at build time |
| `docker history <image>` | Inspect image layers and build args |

---
  during build without exposing them
- `.env` files with `docker compose` — 
  manage build args across environments
