# Day 07 — Docker: containerising an application

**Platform:** KillerCoda  
**Date:** April 2026  
**Topic:** Docker fundamentals — build, run and test a real application

---

## What does containerising an application mean?

Containerising means packaging your application 
and all its dependencies into a Docker image 
so it runs consistently everywhere — on any 
machine, any cloud, any environment.

This is the core workflow every DevOps engineer 
does daily:

```
Source code + Dockerfile
        │
        ▼
   docker build        ← create the image
        │
        ▼
   docker run          ← start the container
        │
        ▼
   curl localhost      ← verify it works
```

---

## Commands practised

```bash
# Build the image
docker build -t app/v1 /root

# Run the container
docker run -d \
  --name app \
  -p 3000:3000 \
  app/v1

# Verify container is running
docker ps

# Send GET request to localhost:3000
curl localhost:3000
```

---

## Full solution — step by step

### Step 1 — inspect the Dockerfile first

```bash
cat /root/Dockerfile
# Always read the Dockerfile before building
# Understand what base image, ports and commands are used
```

### Step 2 — build the image

```bash
docker build -t app/v1 /root
# -t app/v1   = tag/name the image as app/v1
# /root       = build context (where Dockerfile lives)

# Output during build:
# Step 1/N : FROM ...
# Step 2/N : WORKDIR ...
# Step 3/N : COPY ...
# Step 4/N : RUN ...
# Successfully built abc123def456
# Successfully tagged app/v1:latest
```

### Step 3 — verify image was created

```bash
docker image ls
# Output:
# REPOSITORY   TAG      IMAGE ID       SIZE
# app/v1       latest   abc123def456   50MB
```

### Step 4 — run the container

```bash
docker run -d \
  --name app \
  -p 3000:3000 \
  app/v1
# -d           = detached, runs in background
# --name app   = name the container "app"
# -p 3000:3000 = map host port 3000 to container port 3000
# app/v1       = image to use
```

### Step 5 — verify container is running

```bash
docker ps
# Output:
# CONTAINER ID  IMAGE   PORTS                    NAMES
# abc123        app/v1  0.0.0.0:3000->3000/tcp   app
#                       ^^^^^^^^^^^^^^^^^^^^^^
#                       port mapping confirmed
```

### Step 6 — make request to localhost:3000

```bash
curl localhost:3000
# Output: response from the application
# Confirms container is running and serving traffic
```

---

## Understanding the full workflow

```
/root/
├── Dockerfile        ← instructions to build image
├── app.js            ← application source code
├── package.json      ← dependencies
└── ...

docker build -t app/v1 /root
# Docker reads Dockerfile
# Executes each instruction as a layer
# Packages everything into image app/v1

docker run -d --name app -p 3000:3000 app/v1
# Docker creates container from image
# Maps host port 3000 to container port 3000
# Starts application inside container

curl localhost:3000
# Traffic flows: host:3000 → container:3000 → app
```

---

## Image naming convention — app/v1

```bash
# Format: <name>/<version> or <registry>/<name>:<tag>

app/v1          # local image — name/version format
app/v2          # next version
myrepo/app:v1   # registry/name:tag format

# In this lesson app/v1 means:
# app  = application name
# v1   = version 1
```

---

## Typical Dockerfile for a Node.js app

```dockerfile
# What /root/Dockerfile likely contains:
FROM node:alpine

WORKDIR /app

COPY package*.json .
RUN npm install

COPY . .

EXPOSE 3000

CMD ["node", "app.js"]
```

---

## Key takeaways for DevOps

- `docker build → docker run → curl` is the 
  fundamental DevOps container workflow — 
  memorise this sequence
- Always run `docker ps` after `docker run -d` 
  to confirm the container started successfully — 
  a container can exit immediately if the app 
  crashes on startup
- Check logs if the container is not running:

```bash
docker logs app
# Shows stdout/stderr from inside the container
# First place to look when something goes wrong
```

- Port `3000` is the standard port for Node.js 
  applications — you will see it constantly 
  in real projects

---

## Debugging if something goes wrong

```bash
# Container not showing in docker ps?
docker ps -a
# Shows ALL containers including stopped ones
# Check STATUS column

# See why container stopped
docker logs app

# Check if port is already in use
ss -tlnp | grep 3000

# Remove and retry
docker rm -f app
docker run -d --name app -p 3000:3000 app/v1
```

---

## Real DevOps CI/CD connection

```bash
# This lesson is step 3 of a full CI/CD pipeline:

# Step 1 — developer pushes code to GitHub
git push origin main

# Step 2 — Jenkins/GitHub Actions triggers build
docker build -t app/v1 .

# Step 3 — run and test (what we did today)
docker run -d --name app -p 3000:3000 app/v1
curl localhost:3000

# Step 4 — tag and push to registry
docker tag app/v1 registry.company.com/app:v1
docker push registry.company.com/app:v1

# Step 5 — deploy to production
docker pull registry.company.com/app:v1
docker run -d -p 3000:3000 registry.company.com/app:v1
```

---

## New commands learned today

| Command | Purpose |
|---------|---------|
| `docker build -t <name> <path>` | Build image from Dockerfile |
| `docker run -d --name <n> -p <h>:<c> <img>` | Run container detached with name and port |
| `docker ps` | Verify container is running |
| `docker logs <container>` | View container output and errors |
| `docker ps -a` | Show all containers including stopped |
| `curl localhost:<port>` | Test application is serving traffic |

---
