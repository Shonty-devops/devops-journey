# Day 04 — Docker: multi-stage builds

**Platform:** KillerCoda  
**Date:** April 2026  
**Topic:** Docker best practices — reducing image size with multi-stage builds

---

## What is a multi-stage build?

A multi-stage build uses multiple `FROM` statements 
in a single Dockerfile. Each `FROM` starts a new 
stage. You build your application in one stage 
and copy ONLY the final binary into a clean, 
minimal final image.

This solves one of the biggest problems in Docker:
**bloated images full of build tools that are 
not needed at runtime.**

---

## The problem — single stage build

```dockerfile
# Single stage — everything ends up in the image
FROM golang:1.21
WORKDIR /app
COPY . .
RUN go build -o server .
# Final image contains:
# - entire Go compiler (~800MB)
# - source code
# - build cache
# - only then: your tiny binary (~10MB)
# Total image size: ~850MB for a 10MB binary!
```

---

## The solution — multi-stage build

```dockerfile
# Stage 1 — BUILD stage (temporary, thrown away)
FROM golang:1.21 AS builder
WORKDIR /app
COPY . .
RUN go build -o server .

# Stage 2 — FINAL stage (this becomes the image)
FROM scratch
COPY --from=builder /app/server /server
ENTRYPOINT ["/server"]
```

Only the compiled binary is copied into the 
final image. The entire Go compiler and source 
code are discarded.

---

## Step by step — what was done

```bash
# Step 1 — check server-1 image (single stage)
docker image ls server-1
# Output: server-1   latest   abc123   ~850MB
# Large because it contains the full build environment

# Step 2 — build server-2 with multi-stage
docker build -t server-2 /root

# Step 3 — compare image sizes
docker image ls
# Output:
# REPOSITORY   TAG      IMAGE ID       SIZE
# server-1     latest   abc123def456   850MB  ← bloated
# server-2     latest   def456abc123   10MB   ← tiny!
```

---

## Understanding `scratch`

```dockerfile
FROM scratch
```

`scratch` is a special Docker keyword — it is 
not an image. It means:

```
Start from absolutely nothing
No OS
No shell
No utilities
No package manager
Just your binary and nothing else
```

| Base image | Size | Contains |
|-----------|------|----------|
| `ubuntu` | ~80MB | Full Ubuntu OS |
| `alpine` | ~5MB | Minimal Linux |
| `scratch` | 0MB | Absolutely nothing |

`scratch` produces the smallest possible image — 
perfect for compiled binaries that have no 
external dependencies.

---

## Full multi-stage Dockerfile explained

```dockerfile
# ── STAGE 1: builder ──────────────────────────
FROM golang:1.21 AS builder
# AS builder gives this stage a name to reference later

WORKDIR /app
# Set working directory inside build container

COPY . .
# Copy source code into build container

RUN go build -o server .
# Compile the Go binary
# -o server = name the output binary "server"
# This stage is TEMPORARY — discarded after build

# ── STAGE 2: final image ──────────────────────
FROM scratch
# Start from absolute zero — no OS, nothing

COPY --from=builder /app/server /server
# ONLY copy the compiled binary from stage 1
# --from=builder references the named build stage
# Everything else from stage 1 is thrown away

ENTRYPOINT ["/server"]
# Run the binary when container starts
```

---

## Size comparison — server-1 vs server-2

```
server-1 (single stage):
┌──────────────────────────────────┐
│  Go compiler          (~800MB)   │
│  Standard library     (~50MB)    │
│  Source code          (~1MB)     │
│  Build cache          (~20MB)    │
│  Compiled binary      (~10MB)    │
└──────────────────────────────────┘
Total: ~880MB

server-2 (multi-stage with scratch):
┌──────────────────────────────────┐
│  Compiled binary      (~10MB)    │
└──────────────────────────────────┘
Total: ~10MB

Reduction: ~98% smaller image
```

---

## Multiple stages — advanced pattern

```dockerfile
# Stage 1 — dependencies
FROM node:20 AS deps
WORKDIR /app
COPY package*.json .
RUN npm ci

# Stage 2 — build
FROM node:20 AS builder
WORKDIR /app
COPY --from=deps /app/node_modules .
COPY . .
RUN npm run build

# Stage 3 — production image
FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
# Only built static files — no node_modules, no source
```

---

## Key takeaways for DevOps

- Multi-stage builds are a Docker best practice — 
  always use them for compiled languages 
  (Go, Java, Rust, C++)
- Smaller images mean:
  - Faster pull times in CI/CD pipelines
  - Less storage cost in registries
  - Smaller attack surface — fewer vulnerabilities
  - Faster container startup times
- `scratch` is ideal for Go and Rust binaries 
  that compile to self-contained executables
- For interpreted languages (Python, Node.js) 
  use `alpine` as final stage instead of `scratch`
- In Kubernetes, smaller images mean faster 
  pod startup — critical for auto-scaling

---

## When to use which base for final stage

| Language | Build stage | Final stage | Reason |
|----------|------------|-------------|--------|
| Go | `golang:alpine` | `scratch` | Self-contained binary |
| Rust | `rust:alpine` | `scratch` | Self-contained binary |
| Java | `maven` | `eclipse-temurin:jre` | Needs JRE to run |
| Python | `python` | `python:alpine` | Needs interpreter |
| Node.js | `node` | `node:alpine` | Needs runtime |
| React/Vue | `node` | `nginx:alpine` | Only static files needed |

---

## New commands learned today

| Command | Purpose |
|---------|---------|
| `FROM <image> AS <name>` | Start a named build stage |
| `COPY --from=<stage> <src> <dest>` | Copy files from a previous stage |
| `FROM scratch` | Start from an empty base image |
| `docker image ls` | Compare image sizes after build |
