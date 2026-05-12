# Day 5 — Docker: host network

**Platform:** KillerCoda  
**Date:** April 2026  
**Topic:** Docker networking — host network driver

---

## What is the host network?

The host network removes all network isolation 
between the container and the host machine. 
The container shares the host's network stack 
directly — no bridge, no NAT, no virtual network.

```
Bridge network:                Host network:
┌─────────────────┐            ┌─────────────────┐
│   Host machine  │            │   Host machine  │
│   ┌───────────┐ │            │                 │
│   │ Container │ │            │   Container     │
│   │ 172.17.x.x│ │            │   shares host   │
│   └─────┬─────┘ │            │   network       │
│         │ NAT   │            │   directly      │
│   host IP:port  │            │   (same IP)     │
└─────────────────┘            └─────────────────┘
```

---

## Commands practised

```bash
# Remove existing app-1 container
docker rm -f app-1

# Run new app-1 on host network
docker run -d \
  --name app-1 \
  --network host \
  -v /root/app-1:/usr/share/nginx/html \
  nginx:alpine

# Verify container is running
docker ps

# Make request to localhost:80
curl localhost:80
# or
wget -qO- localhost:80
```

---

## Full solution — step by step

### Step 1 — remove existing app-1

```bash
docker rm -f app-1
# -f = force remove even if container is running
# This stops and removes app-1 in one command
```

### Step 2 — start new app-1 on host network

```bash
docker run -d \
  --name app-1 \
  --network host \
  -v /root/app-1:/usr/share/nginx/html \
  nginx:alpine
# --network host = attach to host network directly
# No -p port mapping needed — container uses host ports
```

### Step 3 — verify container is running

```bash
docker ps
# Output:
# CONTAINER ID  IMAGE         NAMES   PORTS
# abc123        nginx:alpine  app-1
# Notice: no port mapping shown — host network bypasses this
```

### Step 4 — make request to localhost:80

```bash
curl localhost:80
# or
wget -qO- localhost:80
# Output: contents of /root/app-1/index.html
# nginx inside container serves directly on host port 80
```

---

## Why no `-p` port mapping is needed

```bash
# Bridge network — needs port mapping:
docker run -d -p 8080:80 nginx:alpine
# host:8080 → container:80 (NAT translation)

# Host network — no port mapping needed:
docker run -d --network host nginx:alpine
# container binds directly to host port 80
# localhost:80 works immediately
```

With host network, the container IS the host 
from a networking perspective.

---

## Host network — what changes

```
Bridge mode:
Host IP: 192.168.1.100
Container IP: 172.17.0.2 (separate)
Access: curl 192.168.1.100:8080 → maps to container:80

Host mode:
Host IP: 192.168.1.100
Container IP: 192.168.1.100 (same as host!)
Access: curl localhost:80 → directly container port 80
```

---

## Comparison — all three network drivers

| Feature | bridge | host | none |
|---------|--------|------|------|
| Container IP | Separate | Same as host | None |
| Port mapping needed | Yes | No | No |
| Network isolation | Yes | No | Complete |
| DNS by name | Custom only | N/A | N/A |
| Performance | Slight overhead | Best | N/A |
| Security | Better | Lower | Maximum |
| Use case | Most apps | High performance | Batch jobs |

---

## Key takeaways for DevOps

- Host network gives best network performance — 
  no NAT overhead, no bridge translation
- Trade-off is reduced isolation — container 
  can access and bind to ANY host port
- Host network is commonly used for:
  - Network monitoring tools that need full 
    host network visibility
  - High-performance applications where NAT 
    overhead is unacceptable
  - Tools like `prometheus node-exporter` that 
    need to see host network metrics
- On Linux, host network works perfectly. 
  On Docker Desktop (Mac/Windows), host network 
  behaves differently due to VM layer
- Never use host network for databases or 
  sensitive services — isolation is important 
  for security

---

## Real DevOps example — prometheus node exporter

```bash
# Node exporter needs host network to collect
# host-level metrics — bridge would hide them

docker run -d \
  --name node-exporter \
  --network host \
  -v /proc:/host/proc:ro \
  -v /sys:/host/sys:ro \
  prom/node-exporter

# Now accessible directly on host:
curl localhost:9100/metrics
```

---

## New commands learned today

| Command | Purpose |
|---------|---------|
| `docker rm -f <container>` | Force remove a running container |
| `docker run --network host` | Attach container to host network |
| `curl localhost:<port>` | Make HTTP request to localhost |
| `wget -qO- localhost:<port>` | Alternative HTTP request tool |

---
