# Day 06 — Docker: publish port on specific host IP

**Platform:** KillerCoda  
**Date:** April 2026  
**Topic:** Docker networking — binding ports to specific host interfaces

---

## What is binding to a specific host IP?

By default, `-p 80:80` binds to ALL network 
interfaces on the host (`0.0.0.0`), meaning 
anyone on any network can reach the container.

Binding to a specific IP like `127.0.0.1` 
restricts access to ONLY that interface — 
in this case, only localhost can reach it, 
not external networks.

---

## Commands practised

```bash
# Run app-2 bound to 127.0.0.1:81
docker run -d \
  --name app-2 \
  -v /root/app-2:/usr/share/nginx/html \
  -p 127.0.0.1:81:80 \
  nginx:alpine

# Verify port mapping
docker ps

# Send GET request to localhost:81
curl localhost:81
# or
wget -qO- localhost:81
```

---

## Full solution — step by step

### Step 1 — run app-2 with specific IP binding

```bash
docker run -d \
  --name app-2 \
  -v /root/app-2:/usr/share/nginx/html \
  -p 127.0.0.1:81:80 \
  nginx:alpine
# -p 127.0.0.1:81:80 means:
# bind to host IP 127.0.0.1 (localhost only)
# host port 81
# container port 80
```

### Step 2 — verify port mapping

```bash
docker ps
# Output:
# CONTAINER ID  IMAGE         PORTS                      NAMES
# abc123        nginx:alpine  127.0.0.1:81->80/tcp       app-2
#                             ^^^^^^^^^^^^^^^^
#                             bound to 127.0.0.1 only
#                             NOT 0.0.0.0 (all interfaces)
```

### Step 3 — send GET request to localhost:81

```bash
curl localhost:81
# Output: contents of /root/app-2/index.html
# Works because we are ON the host machine (127.0.0.1)

# From external machine this would FAIL:
# curl 192.168.1.100:81  → Connection refused
# Because it is bound to 127.0.0.1 only
```

---

## Understanding the three-part `-p` flag

```bash
-p <host-ip>:<host-port>:<container-port>

# Full format with IP:
-p 127.0.0.1:81:80
#  ─────────  ──  ──
#  host IP    │   container port
#             host port

# Short format (no IP = binds to all interfaces):
-p 81:80
# equivalent to:
-p 0.0.0.0:81:80
```

---

## 0.0.0.0 vs 127.0.0.1 — key difference

```
0.0.0.0:80 (default -p 80:80):
┌─────────────────────────────────────┐
│ Host machine                        │
│                                     │
│ eth0: 192.168.1.100:80 ──┐          │
│ eth1: 10.0.0.1:80 ───────┼─► nginx │
│ lo:   127.0.0.1:80 ──────┘          │
│                                     │
│ Accessible from ANYWHERE            │
└─────────────────────────────────────┘

127.0.0.1:81 (specific IP binding):
┌─────────────────────────────────────┐
│ Host machine                        │
│                                     │
│ eth0: 192.168.1.100:81 ──✗          │
│ eth1: 10.0.0.1:81 ───────✗          │
│ lo:   127.0.0.1:81 ──────►  nginx  │
│                                     │
│ Accessible from LOCALHOST ONLY      │
└─────────────────────────────────────┘
```

---

## Day 12 vs Day 13 — comparison

| | Day 12 | Day 13 |
|--|--------|--------|
| Command | `-p 80:80` | `-p 127.0.0.1:81:80` |
| Binds to | `0.0.0.0` (all) | `127.0.0.1` (localhost) |
| Shown in docker ps | `0.0.0.0:80->80/tcp` | `127.0.0.1:81->80/tcp` |
| External access | Yes | No |
| localhost access | Yes | Yes |
| Use case | Public services | Internal/admin services |

---

## Common host IP bindings

```bash
# Bind to localhost only — internal access
-p 127.0.0.1:81:80

# Bind to all interfaces — public access (default)
-p 0.0.0.0:81:80
-p 81:80              # shorthand, same result

# Bind to specific network interface — LAN only
-p 192.168.1.100:81:80
# only accessible from your local network

# Bind to specific docker interface
-p 172.17.0.1:81:80
```

---

## Key takeaways for DevOps

- Binding to `127.0.0.1` is a security best 
  practice for services that should NOT be 
  publicly accessible — admin panels, internal 
  APIs, monitoring dashboards
- Always check `docker ps` and look at the 
  PORTS column carefully:
  - `0.0.0.0:80->80/tcp` = publicly accessible
  - `127.0.0.1:80->80/tcp` = localhost only
- In production, databases should NEVER be 
  bound to `0.0.0.0` — always use `127.0.0.1` 
  or keep them on internal Docker networks only
- This same concept applies in Kubernetes — 
  Services can be ClusterIP (internal only) 
  or LoadBalancer (public)

---

## Real DevOps security example

```bash
# WRONG — database exposed to public internet
docker run -d \
  --name postgres \
  -p 5432:5432 \        # 0.0.0.0 — anyone can connect!
  postgres:alpine

# RIGHT — database only accessible from localhost
docker run -d \
  --name postgres \
  -p 127.0.0.1:5432:5432 \   # localhost only
  postgres:alpine

# BEST — no port mapping at all
# use Docker network for container-to-container comms
docker run -d \
  --name postgres \
  --network backend-net \    # no -p needed
  postgres:alpine
```

---

## Two containers running simultaneously

```bash
# app-1 from Day 12 — public on port 80
docker ps | grep app-1
# 0.0.0.0:80->80/tcp   app-1

# app-2 from Day 13 — localhost only on port 81
docker ps | grep app-2
# 127.0.0.1:81->80/tcp  app-2

# Both nginx containers running:
curl localhost:80   # → app-1 content
curl localhost:81   # → app-2 content
```

---

## New commands learned today

| Command | Purpose |
|---------|---------|
| `docker run -p <ip>:<host-port>:<container-port>` | Bind to specific host IP and port |
| `docker ps` | Verify exact IP binding in PORTS column |
| `curl localhost:<port>` | Test localhost-bound port |

---

## What's next
- `docker port <container>` — list all port mappings 
  for a container
- `EXPOSE` in Dockerfile — document container ports
- `docker run -P` — auto-map all EXPOSE'd ports
- Kubernetes ClusterIP vs NodePort vs LoadBalancer — 
  production port exposure strategies
