# Day 14 — Docker: accessing container ports from another host

**Platform:** KillerCoda  
**Date:** April 2026  
**Topic:** Docker networking — testing port accessibility from remote hosts

---

## What does this lesson demonstrate?

This lesson proves the real-world difference 
between `0.0.0.0` and `127.0.0.1` bindings 
from the perspective of an EXTERNAL machine.

We SSH into a second host (`node01`) and try 
to reach both containers running on `controlplane`:

- `app-1` on port `80` — bound to `0.0.0.0` (public)
- `app-2` on port `81` — bound to `127.0.0.1` (localhost only)

---

## Commands practised

```bash
# Step 1 — SSH into node01 from controlplane
ssh node01

# Step 2 — send GET request to controlplane:80
curl controlplane:80
# or
wget -qO- controlplane:80

# Step 3 — send GET request to controlplane:81
curl controlplane:81
# or
wget -qO- controlplane:81
```

---

## Full solution — step by step

### Step 1 — SSH into node01

```bash
ssh node01
# Connects to the second host in the lab
# You are now operating FROM node01
# node01 is a completely separate machine from controlplane
```

### Step 2 — GET request to controlplane:80

```bash
curl controlplane:80
# Output: contents of app-1 nginx page
# SUCCESS — port 80 is bound to 0.0.0.0
# Accessible from any host on the network
```

### Step 3 — GET request to controlplane:81

```bash
curl controlplane:81
# Output: curl: (7) Failed to connect to controlplane port 81
# FAILS — port 81 is bound to 127.0.0.1
# Only accessible from controlplane itself
# External hosts cannot reach it
```

---

## What this proves

```
controlplane machine:
┌─────────────────────────────────────────┐
│                                         │
│  app-1: 0.0.0.0:80 → container:80      │
│  app-2: 127.0.0.1:81 → container:80    │
│                                         │
└─────────────────────────────────────────┘
        ▲                    ▲
        │                    │
   node01 can          node01 CANNOT
   reach this          reach this
   (public)            (localhost only)


node01 perspective:
┌─────────────────────────────────────────┐
│                                         │
│  curl controlplane:80  → SUCCESS ✓     │
│  curl controlplane:81  → REFUSED ✗     │
│                                         │
└─────────────────────────────────────────┘
```

---

## The full picture — three days of port binding

| Day | Container | Binding | From node01 | From controlplane |
|-----|-----------|---------|-------------|-------------------|
| 12 | app-1 | `0.0.0.0:80->80` | Success ✓ | Success ✓ |
| 13 | app-2 | `127.0.0.1:81->80` | Fails ✗ | Success ✓ |

This table shows exactly WHY binding matters — 
same nginx, same container, different binding, 
completely different accessibility.

---

## SSH basics — commands used

```bash
# Connect to another host by name
ssh node01

# Connect with specific user
ssh root@node01

# Connect with specific port
ssh -p 2222 node01

# Run command on remote host without interactive session
ssh node01 "curl controlplane:80"

# Exit SSH session
exit
# or
Ctrl + D
```

---

## Why this matters in DevOps

```
Production environment example:

controlplane / web server:
┌─────────────────────────────────────────┐
│                                         │
│  nginx:      0.0.0.0:80    → public    │
│  admin UI:   127.0.0.1:8080 → local   │
│  postgres:   127.0.0.1:5432 → local   │
│  prometheus: 127.0.0.1:9090 → local   │
│                                         │
└─────────────────────────────────────────┘

External users → port 80   ✓ (intended)
External users → port 8080 ✗ (blocked — admin panel)
External users → port 5432 ✗ (blocked — database)
External users → port 9090 ✗ (blocked — monitoring)
```

Only the ports that NEED to be public are 
exposed on `0.0.0.0`. Everything else stays 
on `127.0.0.1` — safe and unreachable from outside.

---

## Testing port accessibility — useful commands

```bash
# From remote host — test if port is open
curl -v controlplane:80         # verbose output
curl --connect-timeout 3 controlplane:81  # timeout after 3s
wget -qO- controlplane:80       # alternative to curl
nc -zv controlplane 80          # netcat — just test connection
telnet controlplane 80          # old school port test

# From local machine — check what is listening
ss -tlnp | grep LISTEN          # modern netstat
netstat -tlnp                   # list all listening ports
docker ps                       # check Docker port bindings
```

---

## Key takeaways for DevOps

- Always TEST port accessibility from an external 
  host after deploying — do not assume it works 
  just because it works on localhost
- `ssh node01` then `curl controlplane:port` is 
  exactly how you verify production deployments 
  are accessible from outside
- A port that shows in `docker ps` does NOT 
  automatically mean it is externally accessible — 
  the IP binding determines this
- This exact test is done in real production 
  environments to verify:
  - Load balancers can reach app servers
  - App servers cannot reach databases directly
  - Monitoring ports are not publicly exposed

---

## Real DevOps verification checklist

```bash
# After deploying any service, always verify:

# 1. Check what is running and bound
docker ps
# Look at PORTS column carefully

# 2. Test from same machine
curl localhost:80      # should work
curl localhost:81      # should work

# 3. SSH to another host and test
ssh node01
curl controlplane:80   # should work (0.0.0.0 bound)
curl controlplane:81   # should FAIL (127.0.0.1 bound)

# 4. Confirm exactly what is listening
ss -tlnp | grep nginx
# tcp LISTEN 0.0.0.0:80  → public
# tcp LISTEN 127.0.0.1:81 → localhost only
```

---

## New commands learned today

| Command | Purpose |
|---------|---------|
| `ssh <hostname>` | Connect to remote host via SSH |
| `curl <host>:<port>` | Send HTTP GET to remote host and port |
| `wget -qO- <host>:<port>` | Alternative HTTP GET to remote host |
| `nc -zv <host> <port>` | Test if port is open without HTTP |
| `ss -tlnp` | List all listening ports on current machine |
| `exit` | Exit SSH session and return to original host |

---
  interfaces containers use
- Kubernetes NodePort — expose services on 
  all cluster nodes at a specific port
