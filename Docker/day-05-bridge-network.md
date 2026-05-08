# Day 05 — Docker: bridge network and container communication

**Platform:** KillerCoda  
**Date:** April 2026  
**Topic:** Docker networking — default bridge network and container-to-container communication

---

## What is a Docker network?

Docker networking controls how containers 
communicate with each other and the outside world. 
Docker has several network drivers:

| Driver | Description | Use case |
|--------|-------------|----------|
| `bridge` | Default network, isolated on host | Single host container communication |
| `host` | Shares host network directly | Performance-critical apps |
| `none` | No network access | Fully isolated containers |
| `overlay` | Multi-host networking | Docker Swarm / Kubernetes |

---

## What is the bridge network?

The default bridge network (`docker0`) is 
automatically created when Docker is installed. 
Every container attached to it gets:
- A private IP address (e.g. `172.17.0.2`)
- Ability to communicate with other containers 
  on the same bridge network via IP
- No automatic DNS resolution by hostname 
  (unlike custom bridge networks)

---

## Commands practised

```bash
# Run app-1 container
docker run -d \
  --name app-1 \
  -v /root/app-1:/usr/share/nginx/html \
  nginx:alpine

# Run app-2 container
docker run -d \
  --name app-2 \
  -v /root/app-2:/usr/share/nginx/html \
  nginx:alpine

# Verify both containers are running
docker ps

# Check containers are on default bridge network
docker network inspect bridge

# Get IP address of app-2
docker inspect app-2 | grep IPAddress

# Send GET request from app-1 to app-2 using IP
docker exec app-1 wget -qO- 172.17.0.x

# Send GET request from app-1 to app-2 using hostname
docker exec app-1 wget -qO- app-2
```

---

## Full solution — step by step

### Step 1 — run app-1 container

```bash
docker run -d \
  --name app-1 \
  -v /root/app-1:/usr/share/nginx/html \
  nginx:alpine
# -d                              = run detached (background)
# --name app-1                    = name the container
# -v /root/app-1:/usr/share/...  = mount host directory
# nginx:alpine                    = use nginx alpine image
```

### Step 2 — run app-2 container

```bash
docker run -d \
  --name app-2 \
  -v /root/app-2:/usr/share/nginx/html \
  nginx:alpine
```

### Step 3 — verify both are running

```bash
docker ps
# Output:
# CONTAINER ID  IMAGE         NAMES
# abc123        nginx:alpine  app-2
# def456        nginx:alpine  app-1
```

### Step 4 — confirm bridge network attachment

```bash
docker network inspect bridge
# Output shows both containers listed under "Containers":
# "app-1": { "IPv4Address": "172.17.0.2/16" }
# "app-2": { "IPv4Address": "172.17.0.3/16" }
# Default bridge is used when no --network flag is specified
```

### Step 5 — get app-2 IP address

```bash
docker inspect app-2 | grep IPAddress
# Output: "IPAddress": "172.17.0.3"
```

### Step 6 — send GET request using IP address

```bash
docker exec app-1 wget -qO- 172.17.0.3
# docker exec app-1  = run command inside app-1 container
# wget -qO-          = send GET request, print response
# 172.17.0.3         = app-2 IP address
# Output: contents of /usr/share/nginx/html/index.html from app-2
```

### Step 7 — send GET request using hostname

```bash
docker exec app-1 wget -qO- app-2
# Uses container name as hostname
# Output: contents of app-2 nginx page
```

---

## How volume mounting works

```bash
-v /root/app-1:/usr/share/nginx/html

# Format: -v <host-path>:<container-path>
#
# /root/app-1              = directory ON THE HOST
# /usr/share/nginx/html   = directory INSIDE container
#
# nginx serves files from /usr/share/nginx/html
# So files in /root/app-1 on host are served by nginx
# Changes on host instantly reflect inside container
```

---

## Default bridge network — communication rules

```
Host machine
┌─────────────────────────────────────────┐
│  Docker bridge network (172.17.0.0/16)  │
│                                         │
│  ┌──────────────┐  ┌──────────────┐    │
│  │    app-1     │  │    app-2     │    │
│  │ 172.17.0.2   │  │ 172.17.0.3  │    │
│  └──────┬───────┘  └──────┬───────┘    │
│         │                 │            │
│         └────────┬────────┘            │
│                  │                     │
│             docker0 bridge             │
└─────────────────────────────────────────┘

Communication:
✓ app-1 → app-2 via IP (172.17.0.3)    works
✓ app-1 → app-2 via hostname (app-2)   works
✗ app-1 → external internet            blocked by default
```

---

## Default bridge vs custom bridge network

| Feature | Default bridge | Custom bridge |
|---------|---------------|---------------|
| Automatic DNS by name | No | Yes |
| Container isolation | Shared | Better isolated |
| Created automatically | Yes | `docker network create` |
| Recommended for prod | No | Yes |

```bash
# Custom bridge — recommended for production
docker network create my-network

docker run -d --name app-1 \
  --network my-network \
  -v /root/app-1:/usr/share/nginx/html \
  nginx:alpine

docker run -d --name app-2 \
  --network my-network \
  -v /root/app-2:/usr/share/nginx/html \
  nginx:alpine

# On custom network, DNS works automatically
docker exec app-1 wget -qO- app-2   # works reliably
```

---

## Key takeaways for DevOps

- Default bridge network is fine for testing 
  but custom bridge is recommended for production
- Volume mounts (`-v`) are how you persist data 
  and inject content into containers — used 
  constantly for nginx, databases, and config files
- `docker exec <container> <command>` lets you 
  run any command inside a running container — 
  essential for debugging
- `docker network inspect bridge` is your first 
  tool when debugging container connectivity issues
- In Kubernetes, networking is handled differently — 
  pods communicate via Services, not IPs directly

---

## Real DevOps example — nginx with custom content

```bash
# Host directory structure:
/root/app-1/index.html   ← "Hello from App 1"
/root/app-2/index.html   ← "Hello from App 2"

# Mount into nginx containers:
docker run -d --name app-1 \
  -v /root/app-1:/usr/share/nginx/html \
  nginx:alpine

# Verify content is served:
docker exec app-1 wget -qO- localhost
# Output: Hello from App 1

docker exec app-1 wget -qO- app-2
# Output: Hello from App 2
```

---

## New commands learned today

| Command | Purpose |
|---------|---------|
| `docker run -d` | Run container in detached/background mode |
| `docker run --name <n>` | Assign a name to the container |
| `docker run -v <host>:<container>` | Mount host directory into container |
| `docker network inspect bridge` | Inspect default bridge network |
| `docker inspect <container> \| grep IPAddress` | Get container IP address |
| `docker exec <container> wget -qO- <target>` | Send GET request from inside container |

---
