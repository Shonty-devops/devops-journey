# Day 02 — Docker: push image to registry

**Platform:** KillerCoda  
**Date:** April 2026  
**Topic:** Docker fundamentals — tagging and pushing images to a registry

---

## What is a Docker registry?

A Docker registry is a storage and distribution 
system for Docker images. Think of it like GitHub 
but for Docker images instead of code.

| Registry type | Example | Used for |
|--------------|---------|----------|
| Public registry | Docker Hub | Open source, public images |
| Private registry | AWS ECR, GCR | Company private images |
| Local registry | `local-registry:5000` | Testing, air-gapped environments |

In this lesson we used a **local registry** running 
on port `5000` — a private registry running 
inside the lab environment itself.

---

## Commands practised

| Command | What it does |
|---------|-------------|
| `Ctrl + C` | Stop/exit a running container in terminal |
| `docker tag pinger local-registry:5000/pinger` | Tag existing image with registry destination |
| `docker image ls` | List all local Docker images and their tags |
| `docker push local-registry:5000/pinger` | Push tagged image to the local registry |

---

## Step by step — what was done

```bash
# Step 1 — exit the running container
Ctrl + C
# Stops the running pinger container in terminal

# Step 2 — tag the image for the registry
docker tag pinger local-registry:5000/pinger
# This does NOT create a new image
# It adds an additional tag/alias to the same image

# Step 3 — confirm both tags exist on same image
docker image ls
# Output:
# REPOSITORY                    TAG      IMAGE ID       SIZE
# pinger                        latest   abc123def456   10MB
# local-registry:5000/pinger   latest   abc123def456   10MB
#                                        ^ same IMAGE ID = same image

# Step 4 — push image to local registry
docker push local-registry:5000/pinger
# Docker uploads the image layers to local-registry:5000
```

---

## Understanding `docker tag`

```bash
docker tag <source> <destination>

# Format of destination:
# <registry-host>:<port>/<image-name>:<tag>

docker tag pinger local-registry:5000/pinger
#          ^^^^^^ source image (already built)
#                 ^^^^^^^^^^^^^^^^^^^^^^^^^^^^ destination with registry
```

`docker tag` does NOT duplicate the image — 
it simply adds a new name/label pointing to 
the same image layers. This is why both tags 
show the same IMAGE ID in `docker image ls`.

---

## Understanding Docker image naming

```bash
local-registry:5000/pinger
│               │    │
│               │    └── image name
│               └── registry port
└── registry hostname
```

When you run `docker push`, Docker reads the 
image name to determine WHERE to push it:

```bash
docker push pinger
# Pushes to Docker Hub (docker.io) by default

docker push local-registry:5000/pinger
# Pushes to local-registry on port 5000
```

---

## The push process — what happens layer by layer

```bash
$ docker push local-registry:5000/pinger
# Output:
# The push refers to repository [local-registry:5000/pinger]
# a1b2c3d4e5f6: Pushed        ← layer 1 uploaded
# b2c3d4e5f6a1: Pushed        ← layer 2 uploaded
# latest: digest: sha256:abc... size: 123
```

Docker images are made of layers. Each layer is 
pushed separately. If a layer already exists in 
the registry, Docker skips it — this makes 
subsequent pushes much faster.

---

## Key takeaways for DevOps

- `docker tag` is always done BEFORE `docker push` — 
  you must tag with the registry address so Docker 
  knows where to push
- Every image in a CI/CD pipeline goes through 
  this exact flow: `build → tag → push → deploy`
- Port `5000` is the standard default port for 
  a self-hosted Docker registry
- In production you will use:
  - AWS ECR: `123456789.dkr.ecr.region.amazonaws.com/myapp`
  - Google GCR: `gcr.io/project-id/myapp`
  - Docker Hub: `username/myapp`
  - GitLab Registry: `registry.gitlab.com/group/project/myapp`

---

## Real DevOps CI/CD pipeline example

```bash
# Typical image lifecycle in a Jenkins/GitHub Actions pipeline:

# 1. Build the image
docker build -t myapp .

# 2. Tag for the registry
docker tag myapp registry.company.com/myapp:v1.2.0
docker tag myapp registry.company.com/myapp:latest

# 3. Push both tags
docker push registry.company.com/myapp:v1.2.0
docker push registry.company.com/myapp:latest

# 4. Deploy pulls from registry on the server
docker pull registry.company.com/myapp:latest
docker run -d registry.company.com/myapp:latest
```

---

## New commands learned today

| Command | Purpose |
|---------|---------|
| `docker tag <src> <dest>` | Add registry tag to an existing image |
| `docker image ls` | List all local images and tags |
| `docker push <image>` | Upload image to a registry |
| `Ctrl + C` | Stop a running container attached to terminal |

---
