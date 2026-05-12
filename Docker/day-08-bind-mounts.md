# Day 08 — Docker: bind mounts

**Platform:** KillerCoda  
**Date:** April 2026  
**Topic:** Docker storage — using bind mounts to share host directories

---

## What is a bind mount?

A bind mount maps a specific directory or file 
from the HOST machine directly into the container. 
Any changes made on the host instantly appear 
inside the container, and vice versa.

```
Host machine                Container
┌─────────────────┐         ┌─────────────────┐
│                 │         │                 │
│ /root/app-data  │◄───────►│ /usr/share/     │
│ ├── index.html  │  bind   │ nginx/html      │
│ └── style.css   │  mount  │ ├── index.html  │
│                 │         │ └── style.css   │
└─────────────────┘         └─────────────────┘
     changes here = changes there (instantly)
```

---

## Commands practised

```bash
# Start container with bind mount
docker run -d \
  --name sample-app \
  -v /root/app-data:/usr/share/nginx/html \
  -p 80:80 \
  nginx:alpine

# Verify container is running
docker ps

# Send GET request to localhost:80
curl localhost:80

# List files inside container's mounted directory
docker exec sample-app ls /usr/share/nginx/html
```

---

## Full solution — step by step

### Step 1 — start container with bind mount

```bash
docker run -d \
  --name sample-app \
  -v /root/app-data:/usr/share/nginx/html \
  -p 80:80 \
  nginx:alpine
# -d                                  = run in background
# --name sample-app                   = name the container
# -v /root/app-data:/usr/share/...   = bind mount
# -p 80:80                            = map host:80 to container:80
# nginx:alpine                        = image to use
```

### Step 2 — verify container is running

```bash
docker ps
# Output:
# CONTAINER ID  IMAGE         PORTS                NAMES
# abc123        nginx:alpine  0.0.0.0:80->80/tcp   sample-app
```

### Step 3 — send GET request to localhost:80

```bash
curl localhost:80
# Output: contents of /root/app-data/index.html
# nginx serves files from the bind-mounted directory
```

### Step 4 — list files in mounted directory

```bash
docker exec sample-app ls /usr/share/nginx/html
# Output: lists all files from /root/app-data
# because both paths are the same via bind mount

# For more detail:
docker exec sample-app ls -la /usr/share/nginx/html
```

---

## Understanding the -v flag for bind mounts

```bash
-v <host-path>:<container-path>

-v /root/app-data:/usr/share/nginx/html
#  ───────────── ────────────────────────
#  host path     container path
#  (must exist   (created by Docker
#  on host)       if not exists)

# Key rules:
# host path   = absolute path on host machine
# container   = absolute path inside container
# both paths  = must start with /
```

---

## Live update demonstration

```bash
# Update file on HOST
echo '<h1>Updated from host!</h1>' > /root/app-data/index.html

# Instantly visible inside container
docker exec sample-app cat /usr/share/nginx/html/index.html
# Output: <h1>Updated from host!</h1>

# Instantly visible via curl — no rebuild needed!
curl localhost:80
# Output: <h1>Updated from host!</h1>
```

This is the biggest advantage of bind mounts — 
live updates without rebuilding the image.

---

## Bind mount vs COPY in Dockerfile

| | Bind Mount | COPY in Dockerfile |
|--|-----------|-------------------|
| Files source | Host directory | Build context |
| Updates | Instant — no rebuild | Requires rebuild |
| Persistent | Lives on host | Baked into image |
| Use case | Development | Production |
| Portability | Host-dependent | Works anywhere |

```dockerfile
# Production — files baked into image with COPY
FROM nginx:alpine
COPY ./app-data /usr/share/nginx/html

# Development — use bind mount instead
# docker run -v /root/app-data:/usr/share/nginx/html nginx:alpine
# Edit files on host, see changes instantly in browser
```

---

## Three types of Docker storage

```
1. Bind Mount (-v /host/path:/container/path)
   ├── Host controls the directory
   ├── Path must exist on host
   ├── Best for: development, config files
   └── Data lives on host filesystem

2. Named Volume (-v myvolume:/container/path)
   ├── Docker manages the storage
   ├── Survives container removal
   ├── Best for: databases, persistent app data
   └── Data lives in Docker storage area

3. tmpfs Mount (--tmpfs /container/path)
   ├── Stored in host memory only
   ├── Lost when container stops
   ├── Best for: sensitive temp data
   └── Never written to disk
```

---

## Key takeaways for DevOps

- Bind mounts are the standard approach for 
  development environments — edit code on 
  your machine, see changes instantly in container
- In production, prefer named volumes or 
  baking files into the image with COPY
- `docker exec <container> ls <path>` is 
  essential for verifying what is actually 
  inside a running container — use it to 
  debug mounting issues
- If bind mount directory is empty on host, 
  the container directory will also be empty — 
  this overwrites any default files the image 
  provides:

```bash
# nginx:alpine has default index.html in /usr/share/nginx/html
# But if /root/app-data is empty:
docker run -v /root/app-data:/usr/share/nginx/html nginx:alpine
# Container's /usr/share/nginx/html is now EMPTY
# Default nginx page is gone — replaced by empty host dir
```

---

## Real DevOps example — development workflow

```bash
# Local development with live reload
docker run -d \
  --name myapp-dev \
  -v $(pwd)/src:/app/src \
  -v $(pwd)/public:/app/public \
  -p 3000:3000 \
  node:alpine \
  node server.js

# Edit files in ./src on your laptop
# Changes instantly available in container
# No rebuild needed during development

# When ready for production:
docker build -t myapp:v1 .   # COPY files into image
docker push registry/myapp:v1
```

---

## New commands learned today

| Command | Purpose |
|---------|---------|
| `docker run -v <host>:<container>` | Create bind mount |
| `docker exec <container> ls <path>` | List files inside container |
| `docker exec <container> cat <file>` | Read file inside container |
| `$(pwd)` | Current directory — use in -v for relative paths |

---
