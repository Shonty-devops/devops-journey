# Day 12 — Docker: port forwarding and publishing ports

**Platform:** KillerCoda  
**Date:** April 2026  
**Topic:** Docker networking — mapping host ports to container ports

---

## What is port forwarding in Docker?

Port forwarding (publishing) maps a port on the 
HOST machine to a port INSIDE the container. 
This allows external traffic to reach services 
running inside containers.

```
Without port mapping:
┌─────────────────────────────┐
│ Host machine                │
│                             │
│  ┌───────────────────────┐  │
│  │ Container             │  │
│  │ nginx on port 80      │  │
│  └───────────────────────┘  │
│                             │
│ curl localhost:80 → FAILS   │
│ nginx is unreachable        │
└─────────────────────────────┘

With port mapping (-p 80:80):
┌─────────────────────────────┐
│ Host machine                │
│                             │
│  port 80 ──────────────┐   │
│                         ▼   │
│  ┌───────────────────────┐  │
│  │ Container             │  │
│  │ nginx on port 80      │  │
│  └───────────────────────┘  │
│                             │
│ curl localhost:80 → SUCCESS │
└─────────────────────────────┘
```

---

## Commands practised

```bash
# Run app-1 with port mapping
docker run -d \
  --name app-1 \
  -v /root/app-1:/usr/share/nginx/html \
  -p 80:80 \
  nginx:alpine

# Verify container is running with port mapping
docker ps

# Send GET request to localhost:80
curl localhost:80
# or
wget -qO- localhost:80
```

---

## Full solution — step by step

### Step 1 — run app-1 with port mapping

```bash
docker run -d \
  --name app-1 \
  -v /root/app-1:/usr/share/nginx/html \
  -p 80:80 \
  nginx:alpine
# -d                             = run in background
# --name app-1                   = name the container
# -v /root/app-1:/usr/share/...  = mount host directory
# -p 80:80                       = map host:80 to container:80
# nginx:alpine                   = use nginx alpine image
```

### Step 2 — verify port mapping

```bash
docker ps
# Output:
# CONTAINER ID  IMAGE         PORTS                NAMES
# abc123        nginx:alpine  0.0.0.0:80->80/tcp   app-1
#                             ^^^^^^^^^^^^^^^^
#                             confirms port mapping is active
```

### Step 3 — send GET request to localhost:80

```bash
curl localhost:80
# Output: contents of /root/app-1/index.html
# nginx inside container responds through port mapping
```

---

## Understanding the `-p` flag

```bash
-p <host-port>:<container-port>

-p 80:80
# host port 80 → container port 80

-p 8080:80
# host port 8080 → container port 80
# access via localhost:8080 externally
# nginx still listens on 80 internally

-p 127.0.0.1:80:80
# bind to specific host interface only
# only localhost can access, not external IPs
```

---

## Port mapping variations

```bash
# Map same port
docker run -p 80:80 nginx:alpine

# Map different ports — external:internal
docker run -p 8080:80 nginx:alpine
# curl localhost:8080 → nginx on port 80

# Map multiple ports
docker run -p 80:80 -p 443:443 nginx:alpine

# Map to specific interface
docker run -p 127.0.0.1:80:80 nginx:alpine

# Random host port — Docker picks available port
docker run -p 80 nginx:alpine
# docker ps shows: 0.0.0.0:32768->80/tcp

# Publish all exposed ports randomly
docker run -P nginx:alpine
```

---

## Comparing all networking approaches so far

| Approach | Command | Access method | Use case |
|----------|---------|---------------|----------|
| No mapping | `docker run nginx` | Not accessible | Internal only |
| Port mapping | `docker run -p 80:80 nginx` | `localhost:80` | Standard apps |
| Host network | `docker run --network host nginx` | `localhost:80` | High performance |
| Custom bridge | `docker network create` | By container name | Microservices |

---

## How port mapping works internally

```
External request → localhost:80
         │
         ▼
    Host port 80
         │
    Docker NAT/iptables
         │
         ▼
  Container port 80
         │
         ▼
    nginx serves
    /usr/share/nginx/html
```

Docker uses Linux `iptables` rules under the 
hood to redirect traffic from host ports to 
container ports automatically.

---

## Key takeaways for DevOps

- `-p` flag is the standard way to expose 
  containerised services to the outside world
- Format is always `host:container` — easy to 
  remember: outside:inside
- Multiple containers cannot bind to the same 
  host port simultaneously:

```bash
docker run -p 80:80 nginx   # works
docker run -p 80:80 nginx   # FAILS — port 80 already in use
docker run -p 8080:80 nginx # works — different host port
```

- In production Kubernetes, port mapping is 
  handled by Services (NodePort, LoadBalancer) — 
  you will not use `-p` flag directly in K8s
- Always check `docker ps` after running a 
  container to confirm port mapping is active

---

## Real DevOps example — multiple services on different ports

```bash
# Run multiple nginx containers on different host ports
docker run -d --name frontend \
  -v /root/frontend:/usr/share/nginx/html \
  -p 80:80 \
  nginx:alpine

docker run -d --name backend-api \
  -p 3000:3000 \
  myapi:latest

docker run -d --name admin-panel \
  -v /root/admin:/usr/share/nginx/html \
  -p 8080:80 \
  nginx:alpine

# Access each service:
# curl localhost:80    → frontend
# curl localhost:3000  → backend API
# curl localhost:8080  → admin panel
```

---

## New commands learned today

| Command | Purpose |
|---------|---------|
| `docker run -p <host>:<container>` | Map host port to container port |
| `docker run -P` | Randomly map all exposed ports |
| `docker ps` | Verify port mappings are active |
| `curl localhost:<port>` | Test port mapping with HTTP request |

---
- `docker port <container>` — list all port mappings
- Kubernetes NodePort and LoadBalancer Services — 
  production-grade port exposure
