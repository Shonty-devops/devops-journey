# Day 07 — Docker: updating a containerised application

**Platform:** KillerCoda  
**Date:** April 2026  
**Topic:** Docker fundamentals — rebuilding and redeploying updated application

---

## What does updating a containerised app mean?

When application code changes, you cannot simply 
edit files inside a running container — containers 
are immutable. The correct workflow is:

```
Update source code
      │
      ▼
Build new image (app/v2)
      │
      ▼
Stop and remove old container
      │
      ▼
Start new container with new image
      │
      ▼
Verify update is live
```

This immutability is a FEATURE — it guarantees 
every deployment is clean, consistent, and 
reproducible.

---

## Commands practised

```bash
# Append line to index.html
echo '<h2>Some updates for app/v2</h2>' >> /root/index.html

# Verify the change
cat /root/index.html

# Build new image
docker build -t app/v2 /root

# Remove old container
docker rm -f app

# Start new container with updated image
docker run -d \
  --name app \
  -p 3000:3000 \
  app/v2

# Verify update is live
curl localhost:3000
```

---

## Full solution — step by step

### Step 1 — append update to index.html

```bash
echo '<h2>Some updates for app/v2</h2>' >> /root/index.html
# echo  = print the text
# >>    = APPEND to file (single > would overwrite!)
# /root/index.html = target file

# Verify the change was appended correctly
cat /root/index.html
# Last line should now be:
# <h2>Some updates for app/v2</h2>
```

### Step 2 — build updated image as app/v2

```bash
docker build -t app/v2 /root
# -t app/v2 = name new image app/v2
# /root     = build context with updated index.html

# Docker uses cached layers where possible:
# Step 1/N FROM ... → Using cache
# Step 2/N COPY ... → Using cache
# Step 3/N COPY index.html ... → new layer (file changed)
# Successfully tagged app/v2:latest
```

### Step 3 — verify both images exist

```bash
docker image ls
# Output:
# REPOSITORY   TAG      IMAGE ID       SIZE
# app/v2       latest   def456abc123   50MB  ← new
# app/v1       latest   abc123def456   50MB  ← old still exists
```

### Step 4 — remove old container

```bash
docker rm -f app
# -f = force remove even if running
# Stops and removes the app/v1 container in one command
```

### Step 5 — start new container with app/v2

```bash
docker run -d \
  --name app \
  -p 3000:3000 \
  app/v2
# Same name, same port — but now uses app/v2 image
```

### Step 6 — verify update is live

```bash
curl localhost:3000
# Output should now include:
# <h2>Some updates for app/v2</h2>
# Confirms the updated image is serving traffic
```

---

## Understanding `>>` vs `>`

```bash
# >> APPENDS to file — keeps existing content
echo '<h2>update</h2>' >> /root/index.html
# Before: <h1>Hello World</h1>
# After:  <h1>Hello World</h1>
#         <h2>update</h2>

# > OVERWRITES file — destroys existing content
echo '<h2>update</h2>' > /root/index.html
# Before: <h1>Hello World</h1>
# After:  <h2>update</h2>     ← original content GONE
```

Always double-check which operator you need 
before running — `>` has caused many accidents 
in production by overwriting important files.

---

## Docker layer caching during rebuild

```
app/v1 build (Day 15):          app/v2 build (today):
Layer 1: FROM node:alpine        Layer 1: FROM node:alpine    (cached)
Layer 2: WORKDIR /app            Layer 2: WORKDIR /app        (cached)
Layer 3: COPY package.json       Layer 3: COPY package.json   (cached)
Layer 4: RUN npm install         Layer 4: RUN npm install     (cached)
Layer 5: COPY index.html         Layer 5: COPY index.html     (NEW — file changed)
Layer 6: CMD node app.js         Layer 6: CMD node app.js     (rebuilt)

Result: app/v2 builds much faster
because unchanged layers are reused from cache
```

---

## v1 vs v2 — full comparison

| | app/v1 | app/v2 |
|--|--------|--------|
| Image | `app/v1` | `app/v2` |
| index.html | Original content | + `<h2>Some updates for app/v2</h2>` |
| Container name | `app` | `app` (same name, new image) |
| Port | `3000:3000` | `3000:3000` |
| Response | Original page | Updated page |

---

## The immutability principle

```bash
# WRONG — trying to edit inside running container
docker exec app vi /root/index.html
# Even if this works temporarily:
# - change is lost when container restarts
# - other instances of container are not updated
# - no record of what changed or when

# RIGHT — update source, rebuild image, redeploy
echo '<h2>update</h2>' >> /root/index.html
docker build -t app/v2 /root
docker rm -f app
docker run -d --name app -p 3000:3000 app/v2
# Every change is tracked, versioned, reproducible
```

---

## Key takeaways for DevOps

- Container immutability is a core principle — 
  never edit files inside running containers 
  in production
- Always build a new image version for every 
  code change — this creates a full audit trail
- Old images should be kept for rollback:

```bash
# If app/v2 has a bug — rollback is instant
docker rm -f app
docker run -d --name app -p 3000:3000 app/v1
# Back to v1 in seconds
```

- In production, use semantic versioning for 
  image tags — `app:1.0.0`, `app:1.0.1` — 
  not just v1, v2
- `>>` vs `>` is one of the most important 
  distinctions in shell scripting — wrong 
  operator has caused production incidents

---

## Real DevOps deployment workflow

```bash
# Developer updates code and pushes to git
git add index.html
git commit -m "Add v2 update heading"
git push origin main

# CI/CD pipeline triggers automatically:
# 1. Pull latest code
git pull origin main

# 2. Build new versioned image
VERSION=$(git rev-parse --short HEAD)
docker build -t app:${VERSION} .
docker tag app:${VERSION} app:latest

# 3. Push to registry
docker push registry.company.com/app:${VERSION}

# 4. Deploy new version
docker rm -f app
docker run -d \
  --name app \
  -p 3000:3000 \
  registry.company.com/app:${VERSION}

# 5. Verify
curl localhost:3000
```

---

## New commands learned today

| Command | Purpose |
|---------|---------|
| `echo 'text' >> file` | Append text to file without overwriting |
| `echo 'text' > file` | Overwrite file with text |
| `cat <file>` | Verify file contents after editing |
| `docker build -t <new-version> <path>` | Build updated image with new tag |
| `docker rm -f <container>` | Force remove running container |
| `docker image ls` | Confirm both old and new images exist |

---
