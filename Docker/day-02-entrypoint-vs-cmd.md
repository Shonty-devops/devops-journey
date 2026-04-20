# Day 03 — Docker: ENTRYPOINT vs CMD

**Platform:** KillerCoda  
**Date:** April 2026  
**Topic:** Docker fundamentals — difference between ENTRYPOINT and CMD

---

## What is the difference?

This is one of the most commonly asked Docker 
interview questions. Both `CMD` and `ENTRYPOINT` 
define what runs when a container starts — but 
they behave very differently.

| | CMD | ENTRYPOINT |
|--|-----|-----------|
| Purpose | Default command — easily overridden | Main command — always runs |
| Override via CLI | Yes — just add command after `docker run` | Requires `--entrypoint` flag |
| Used for | Default arguments or fallback commands | Fixed executable that must always run |

---

## Commands practised

| Command | What it does |
|---------|-------------|
| `vim /root/Dockerfile` | Edit the Dockerfile |
| `docker build -t entrypoint-echo /root` | Build image from /root/Dockerfile |
| `docker inspect entrypoint-echo` | Inspect image — check ENTRYPOINT config |
| `docker run entrypoint-echo` | Run container normally |
| `docker run --entrypoint date entrypoint-echo` | Override ENTRYPOINT with `date` via CLI |
| `docker run entrypoint-echo date` | Set CMD to `date` via CLI (appended to ENTRYPOINT) |

---

## Step by step — what was done

### Step 1 — edit the Dockerfile

```bash
vim /root/Dockerfile
```

Remove the existing CMD line and replace with ENTRYPOINT:

```dockerfile
# BEFORE (with CMD)
FROM alpine
CMD ["echo", "hi, from container!"]

# AFTER (with ENTRYPOINT)
FROM alpine
ENTRYPOINT ["echo", "hi, from container!"]
```

Save and exit: `ESC` → `:wq`

---

### Step 2 — build the image

```bash
docker build -t entrypoint-echo /root
# -t entrypoint-echo  = name the image
# /root               = location of Dockerfile
```

---

### Step 3 — inspect the image

```bash
docker inspect entrypoint-echo
# Look for the "Entrypoint" field in output:
# "Entrypoint": [
#     "echo",
#     "hi, from container!"
# ]
```

---

### Step 4 — run normally

```bash
docker run entrypoint-echo
# Output: hi, from container!
# ENTRYPOINT runs as defined
```

---

### Step 5 — override ENTRYPOINT with `--entrypoint`

```bash
docker run --entrypoint date entrypoint-echo
# Output: Mon Apr 20 10:00:00 UTC 2026
# --entrypoint flag REPLACES the entrypoint completely
# "date" command runs instead of "echo hi, from container!"
```

---

### Step 6 — set CMD to `date` via CLI

```bash
docker run entrypoint-echo date
# Output: echo hi, from container! date
# "date" is APPENDED as argument to ENTRYPOINT
# ENTRYPOINT stays as "echo" — "date" becomes its argument
```

---

## Key concept — how CMD and ENTRYPOINT interact

```
ENTRYPOINT = the executable (fixed)
CMD        = the arguments (flexible/overridable)

docker run entrypoint-echo
→ runs: echo "hi, from container!"

docker run entrypoint-echo date
→ runs: echo date
  (CMD "date" replaces default args, passed to echo)

docker run --entrypoint date entrypoint-echo
→ runs: date
  (ENTRYPOINT itself is replaced)
```

---

## Visual comparison

```
┌─────────────────────────────────────────────┐
│              docker run myimage             │
├─────────────────────────────────────────────┤
│  ENTRYPOINT ["echo"]  +  CMD ["hello"]      │
│  → runs: echo hello                         │
├─────────────────────────────────────────────┤
│  docker run myimage world                   │
│  → runs: echo world  (CMD overridden)       │
├─────────────────────────────────────────────┤
│  docker run --entrypoint date myimage       │
│  → runs: date  (ENTRYPOINT overridden)      │
└─────────────────────────────────────────────┘
```

---

## Two Dockerfile formats — exec vs shell

```dockerfile
# Exec format (recommended — use this)
ENTRYPOINT ["echo", "hi, from container!"]
CMD ["default-argument"]

# Shell format (runs via /bin/sh -c)
ENTRYPOINT echo hi, from container!
CMD default-argument
```

Always use exec format `["command", "arg"]` in 
production — shell format makes signal handling 
unreliable and can cause containers to not 
stop cleanly.

---

## When to use which in DevOps

### Use ENTRYPOINT when:
```dockerfile
# Container has ONE clear purpose
ENTRYPOINT ["nginx", "-g", "daemon off;"]
ENTRYPOINT ["python", "app.py"]
ENTRYPOINT ["java", "-jar", "app.jar"]
```

### Use CMD when:
```dockerfile
# Default behaviour but easily overridable
CMD ["--help"]           # show help by default
CMD ["bash"]             # open shell by default
```

### Use BOTH together (most common pattern):
```dockerfile
# Fixed executable + overridable default args
ENTRYPOINT ["python", "app.py"]
CMD ["--port", "8080"]   # default port, easily changed

# Run with defaults:
docker run myapp
# → python app.py --port 8080

# Override port:
docker run myapp --port 9090
# → python app.py --port 9090
```

---

## Key takeaways for DevOps

- `ENTRYPOINT` defines WHAT the container is — 
  the main process that must always run
- `CMD` defines HOW it runs by default — 
  arguments that can be easily changed
- Using both together is the most flexible and 
  professional pattern in production Dockerfiles
- `--entrypoint` flag in `docker run` is useful 
  for debugging — lets you override and run 
  `bash` or `sh` inside a container that has 
  a fixed entrypoint
- Kubernetes `command:` maps to ENTRYPOINT and 
  `args:` maps to CMD in pod specs

---

## Kubernetes connection

```yaml
# In Kubernetes pod spec:
containers:
  - name: myapp
    image: myapp:v1
    command: ["python"]      # overrides ENTRYPOINT
    args: ["app.py", "--port", "9090"]  # overrides CMD
```

Understanding ENTRYPOINT vs CMD in Docker 
is essential for writing Kubernetes manifests.

---

## New commands learned today

| Command | Purpose |
|---------|---------|
| `docker build -t <name> <path>` | Build image from Dockerfile at path |
| `docker inspect <image>` | View detailed image configuration |
| `docker run --entrypoint <cmd> <image>` | Override ENTRYPOINT at runtime |
| `docker run <image> <cmd>` | Pass CMD argument at runtime |
