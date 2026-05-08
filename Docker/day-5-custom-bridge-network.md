# Day 5 — Docker: custom bridge network

**Platform:** KillerCoda  
**Date:** April 2026  
**Topic:** Docker networking — creating and using custom bridge networks

---

## What is a custom bridge network?

A custom bridge network is a user-defined network 
that you create with `docker network create`. 
Unlike the default bridge, a custom bridge network 
provides automatic DNS resolution — containers 
can reach each other by name, not just IP address.

---

## Default bridge vs custom bridge — recap

| Feature | Default bridge | Custom bridge |
|---------|---------------|---------------|
| Automatic DNS by name | No | Yes |
| Isolation | Shared with all containers | Only attached containers |
| Created by | Docker automatically | `docker network create` |
| Recommended | Testing only | Production |

---

## Commands practised

```bash
# Create custom bridge network
docker network create bridge-network

# Detach app-1 from default bridge
docker network disconnect bridge app-1

# Detach app-2 from default bridge
docker network disconnect bridge app-2

# Attach app-1 to custom bridge-network
docker network connect bridge-network app-1

# Attach app-2 to custom bridge-network
docker network connect bridge-network app-2

# Verify both containers on new network
docker network inspect bridge-network

# Get app-2 IP on new network
docker inspect app-2 | grep IPAddress

# Send GET request from app-1 to app-2 by IP
docker exec app-1 wget -qO- <app-2-ip>

# Send GET request from app-1 to app-2 by hostname
docker exec app-1 wget -qO- app-2
```

---

## Full solution — step by step

### Step 1 — create the custom bridge network

```bash
docker network create bridge-network
# Output: a1b2c3d4e5f6... (network ID)
# Creates a new isolated bridge network
```

### Step 2 — verify network was created

```bash
docker network ls
# Output:
# NETWORK ID     NAME             DRIVER    SCOPE
# abc123def456   bridge           bridge    local  ← default
# def456abc123   bridge-network   bridge    local  ← new one
# xyz789uvw012   host             host      local
# uvw012xyz789   none             null      local
```

### Step 3 — detach containers from default bridge

```bash
docker network disconnect bridge app-1
# Removes app-1 from default bridge network

docker network disconnect bridge app-2
# Removes app-2 from default bridge network
```

### Step 4 — attach containers to custom network

```bash
docker network connect bridge-network app-1
# Connects app-1 to bridge-network

docker network connect bridge-network app-2
# Connects app-2 to bridge-network
```

### Step 5 — verify both containers on new network

```bash
docker network inspect bridge-network
# Output shows under "Containers":
# "app-1": { "IPv4Address": "192.168.x.2/20" }
# "app-2": { "IPv4Address": "192.168.x.3/20" }
# Note: custom networks use different subnet than default bridge
```

### Step 6 — get app-2 IP address

```bash
docker inspect app-2 | grep IPAddress
# Output: "IPAddress": "192.168.x.3"
```

### Step 7 — send GET request by IP address

```bash
docker exec app-1 wget -qO- 192.168.x.3
# Output: contents of app-2 nginx page
```

### Step 8 — send GET request by hostname

```bash
docker exec app-1 wget -qO- app-2
# Works automatically on custom bridge network!
# DNS resolves "app-2" to its IP automatically
# Output: contents of app-2 nginx page
```

---

## How custom bridge DNS works

```
Default bridge network:
┌─────────────────────────────────────┐
│  app-1 (172.17.0.2)                 │
│  tries: wget app-2                  │
│  result: FAILS — no DNS resolution  │
│  must use IP: wget 172.17.0.3       │
└─────────────────────────────────────┘

Custom bridge network:
┌─────────────────────────────────────┐
│  app-1 (192.168.x.2)               │
│  tries: wget app-2                  │
│  Docker DNS resolves "app-2"        │
│  → automatically finds 192.168.x.3 │
│  result: SUCCESS                    │
└─────────────────────────────────────┘
```

---

## Network topology — before and after

```
BEFORE (default bridge):
┌──────────────────────────────────┐
│  bridge (docker0) 172.17.0.0/16  │
│  ┌────────┐      ┌────────┐      │
│  │ app-1  │      │ app-2  │      │
│  │.0.2    │      │ .0.3   │      │
│  └────────┘      └────────┘      │
└──────────────────────────────────┘
No DNS — must use IP to communicate

AFTER (custom bridge):
┌────────────────────────────────────┐
│  bridge-network  192.168.x.0/20   │
│  ┌────────┐       ┌────────┐      │
│  │ app-1  │  DNS  │ app-2  │      │
│  │  .x.2  │◄─────►│  .x.3  │      │
│  └────────┘       └────────┘      │
└────────────────────────────────────┘
DNS enabled — use name OR IP
```

---

## docker network commands — full reference

```bash
# Create network
docker network create <name>
docker network create --driver bridge <name>   # explicit driver
docker network create --subnet 10.0.0.0/24 <name>  # custom subnet

# List all networks
docker network ls

# Inspect a network
docker network inspect <name>

# Connect container to network
docker network connect <network> <container>

# Disconnect container from network
docker network disconnect <network> <container>

# Remove a network (no containers must be attached)
docker network rm <name>

# Remove all unused networks
docker network prune
```

---

## Key takeaways for DevOps

- Always use custom bridge networks in production — 
  never rely on the default bridge for real apps
- Custom bridge networks provide automatic DNS — 
  containers find each other by name, making 
  configs portable and not dependent on IPs
- `docker network connect` and `disconnect` let 
  you move running containers between networks 
  without restarting them
- In microservices architecture, you can have 
  multiple custom networks for isolation:

```bash
# Frontend network — only frontend talks to backend
docker network create frontend-network

# Backend network — only backend talks to database
docker network create backend-network

# backend container joins BOTH networks
docker network connect frontend-network backend
docker network connect backend-network backend

# database only on backend network — isolated from frontend
docker network connect backend-network database
```

---

## Real DevOps example — microservices isolation

```bash
# Create isolated networks
docker network create frontend-net
docker network create backend-net

# Frontend only on frontend-net
docker run -d --name frontend \
  --network frontend-net \
  nginx:alpine

# API on both networks (bridges frontend and backend)
docker run -d --name api \
  --network frontend-net \
  myapi:latest
docker network connect backend-net api

# Database only on backend-net (never exposed to frontend)
docker run -d --name database \
  --network backend-net \
  postgres:alpine

# Result:
# frontend → api    ✓ (same network)
# api → database    ✓ (same network)
# frontend → database ✗ (different networks — isolated!)
```

---

## New commands learned today

| Command | Purpose |
|---------|---------|
| `docker network create <name>` | Create a new custom bridge network |
| `docker network ls` | List all Docker networks |
| `docker network inspect <name>` | View network details and attached containers |
| `docker network connect <net> <container>` | Attach container to network |
| `docker network disconnect <net> <container>` | Detach container from network |
| `docker network rm <name>` | Remove unused network |

---

## What's next
- `docker network create --subnet` — define custom IP range
- `docker compose` — manages networks automatically
- `docker run --network host` — host network mode
- Kubernetes Services — how K8s handles networking
  between pods (replaces Docker networking entirely)
