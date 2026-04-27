# Day 04 — Docker: building specific stages with --target

**Platform:** KillerCoda  
**Date:** April 2026  
**Topic:** Docker best practices — targeting specific stages in multi-stage builds

---

## What is `--target`?

In a multi-stage Dockerfile, `--target` lets you 
build UP TO a specific stage and stop there. 
This means one single Dockerfile can produce 
multiple different images — each built from 
a different target stage.

---

## The problem it solves

```dockerfile
# Without --target, docker build always runs
# ALL stages and produces only the FINAL image.

# But what if you need:
# - a server image
# - a client image
# - a debug image
# - a test image
# All from the same codebase?

# Answer: name your stages and use --target
```

---

## The Dockerfile structure

```dockerfile
# Stage 1 — shared base
FROM alpine AS base
WORKDIR /app
COPY . .

# Stage 2 — server image
FROM base AS server
RUN echo "building server..."
CMD ["./server"]

# Stage 3 — client image  
FROM base AS client
RUN echo "building client..."
CMD ["./client"]
```

Each stage has a name after `AS`. 
`--target` tells Docker which stage to stop at.

---

## Full solution — step by step

```bash
# Step 1 — build the server image using --target
docker build --target server -t run-server /root
# --target server  = stop at the stage named "server"
# -t run-server    = name the resulting image run-server
# /root            = location of Dockerfile

# Step 2 — build the client image using --target
docker build --target client -t run-client /root
# --target client  = stop at the stage named "client"
# -t run-client    = name the resulting image run-client

# Step 3 — verify both images were created
docker image ls
# Output:
# REPOSITORY    TAG      IMAGE ID       SIZE
# run-server    latest   abc123def456   10MB
# run-client    latest   def456abc123   10MB
# Both built from the SAME Dockerfile
```

---

## How `--target` works visually

```
Dockerfile stages:
┌─────────────┐
│    base     │  Stage 1
└──────┬──────┘
       │
  ┌────┴────┐
  ▼         ▼
┌──────┐ ┌──────┐
│server│ │client│  Stage 2 and 3
└──────┘ └──────┘

docker build --target server
→ runs: base → server
→ stops here, ignores client stage

docker build --target client
→ runs: base → client
→ stops here, ignores server stage

docker build (no target)
→ runs ALL stages
→ produces final stage image only
```

---

## Comparison — Day 06 vs Day 07

| Feature | Day 06 multi-stage | Day 07 --target |
|---------|-------------------|-----------------|
| Purpose | Reduce image size | Build multiple images from one Dockerfile |
| How | COPY --from= | --target flag |
| Output | One final image | Multiple targeted images |
| Use case | Remove build tools | Server + client + test images |

---

## Real DevOps example — one Dockerfile, four targets

```dockerfile
FROM node:20-alpine AS base
WORKDIR /app
COPY package*.json .
RUN npm ci

FROM base AS development
COPY . .
CMD ["npm", "run", "dev"]

FROM base AS test
COPY . .
CMD ["npm", "test"]

FROM base AS builder
COPY . .
RUN npm run build

FROM nginx:alpine AS production
COPY --from=builder /app/dist /usr/share/nginx/html

# Build commands:
# docker build --target development -t myapp:dev .
# docker build --target test -t myapp:test .
# docker build --target production -t myapp:prod .
```

---

## Usage in CI/CD pipelines

```bash
# Jenkins / GitHub Actions pipeline example:

# Build and test
docker build --target test -t myapp:test .
docker run myapp:test

# If tests pass, build production image
docker build --target production -t myapp:prod .
docker tag myapp:prod registry.company.com/myapp:v1.0.0
docker push registry.company.com/myapp:v1.0.0
```

This is extremely common in professional CI/CD 
pipelines — test stage runs first, production 
image only builds if tests pass.

---

## Key takeaways for DevOps

- One Dockerfile per service is cleaner than 
  multiple Dockerfiles — `--target` makes this possible
- Layer caching is shared between targets — 
  building `run-server` after `run-client` is 
  faster because the `base` stage is already cached
- `--target` is heavily used in monorepos where 
  server and client live in the same repository
- In Kubernetes, you might deploy `run-server` 
  and `run-client` as separate pods but maintain 
  them in a single Dockerfile — clean and efficient

---

## New commands learned today

| Command | Purpose |
|---------|---------|
| `FROM <image> AS <name>` | Name a stage for targeting |
| `docker build --target <stage> -t <name> <path>` | Build up to a specific stage |
| `docker image ls` | Verify multiple images were created |

---
