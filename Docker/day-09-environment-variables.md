# Day 09 — Docker: environment variables with ENV

**Platform:** KillerCoda  
**Date:** April 2026  
**Topic:** Docker fundamentals — defining and verifying environment variables

---

## What are environment variables in Docker?

Environment variables pass configuration into 
containers without hardcoding values into 
application code. They are the standard way 
to configure containerised applications 
for different environments.

```
Same image → different behaviour via ENV:

docker run -e DB_HOST=dev.db myapp     → connects to dev
docker run -e DB_HOST=prod.db myapp    → connects to prod

No code change. No rebuild. Just config.
```

---

## Commands practised

```bash
# Create the Dockerfile
cat > /root/Dockerfile << 'EOF'
FROM nginx:alpine
ENV key1=value1
EOF

# Verify Dockerfile content
cat /root/Dockerfile

# Build the image
docker build -t sample-image /root

# Run the container
docker run -d --name sample-app sample-image

# Confirm environment variable inside container
docker exec sample-app env | grep key1
# or
docker exec sample-app printenv key1
```

---

## Full solution — step by step

### Step 1 — create the Dockerfile

```bash
cat > /root/Dockerfile << 'EOF'
FROM nginx:alpine
ENV key1=value1
EOF

# Or using vim:
vim /root/Dockerfile
```

Dockerfile contents:
```dockerfile
FROM nginx:alpine
ENV key1=value1
```

### Step 2 — verify Dockerfile was created

```bash
cat /root/Dockerfile
# Output:
# FROM nginx:alpine
# ENV key1=value1
```

### Step 3 — build the image

```bash
docker build -t sample-image /root
# -t sample-image = name the image
# /root           = location of Dockerfile

# Output:
# Step 1/2 : FROM nginx:alpine
# Step 2/2 : ENV key1=value1
# Successfully tagged sample-image:latest
```

### Step 4 — run the container

```bash
docker run -d --name sample-app sample-image
# -d               = run in background
# --name sample-app = name the container
# sample-image     = image to use
# No port mapping needed — just verifying ENV
```

### Step 5 — confirm environment variable

```bash
# Method 1 — list all env vars and filter
docker exec sample-app env | grep key1
# Output: key1=value1

# Method 2 — print specific variable
docker exec sample-app printenv key1
# Output: value1

# Method 3 — echo the variable
docker exec sample-app sh -c 'echo $key1'
# Output: value1

# Method 4 — inspect image config
docker inspect sample-app | grep -A 5 "Env"
# Output:
# "Env": [
#     "key1=value1",
#     "PATH=/usr/local/sbin:..."
# ]
```

---

## The ENV instruction

```dockerfile
# Single variable
ENV key1=value1

# Multiple variables — one ENV instruction
ENV key1=value1 key2=value2 key3=value3

# Multiple variables — separate ENV instructions
ENV key1=value1
ENV key2=value2

# With spaces in value
ENV APP_NAME="My Application"

# Using previously defined ENV
ENV BASE_DIR=/app
ENV LOG_DIR=${BASE_DIR}/logs
```

---

## ENV in Dockerfile vs -e at runtime

```bash
# ENV in Dockerfile — baked into image, always present
FROM nginx:alpine
ENV key1=value1        # always key1=value1 unless overridden

# -e at runtime — override or add variables
docker run -e key1=newvalue sample-image
# key1 is now "newvalue" not "value1"

docker run -e key2=value2 sample-image
# key2 added at runtime, key1 still "value1"

# --env-file — load many variables from file
docker run --env-file .env sample-image
```

---

## Environment variable priority

```
Highest priority → runtime -e flag
                   docker run -e key1=override

                ↓ overrides

Middle priority → Dockerfile ENV instruction
                  ENV key1=value1

                ↓ overrides

Lowest priority → default (variable not set)
```

---

## Verifying ENV — all methods compared

```bash
# Inside container — see all variables
docker exec sample-app env

# Inside container — see specific variable
docker exec sample-app printenv key1

# Inside container — echo variable
docker exec sample-app sh -c 'echo $key1'

# From host — inspect container config
docker inspect sample-app | grep -A 10 '"Env"'

# From host — inspect image config
docker inspect sample-image | grep -A 10 '"Env"'
```

---

## Common DevOps environment variables

```dockerfile
# Web application
FROM node:alpine
ENV NODE_ENV=production
ENV PORT=3000
ENV LOG_LEVEL=info

# Database connection
ENV DB_HOST=localhost
ENV DB_PORT=5432
ENV DB_NAME=myapp

# Application config
ENV MAX_CONNECTIONS=100
ENV CACHE_TTL=3600
ENV API_VERSION=v1
```

---

## Security warning — sensitive values in ENV

```dockerfile
# NEVER put secrets in Dockerfile ENV
ENV DB_PASSWORD=secret123      # WRONG
ENV API_KEY=abc123xyz          # WRONG

# These are visible via:
docker history sample-image    # shows all layers
docker inspect sample-app      # shows all env vars
```

For secrets use:
```bash
# Runtime injection — not stored in image
docker run -e DB_PASSWORD=secret sample-app

# Docker secrets (Swarm)
docker secret create db_password ./password.txt

# Kubernetes secrets
kubectl create secret generic db-creds \
  --from-literal=password=secret
```

---

## Key takeaways for DevOps

- `ENV` in Dockerfile sets default configuration 
  that works out of the box — no extra flags needed
- Runtime `-e` flag overrides Dockerfile `ENV` — 
  same image runs differently in dev, staging, prod
- `docker exec <container> env` is your first 
  debugging tool when an app behaves differently 
  than expected — verify the variables it sees
- Never store passwords or API keys in `ENV` 
  instructions — they are visible in image history
- In Kubernetes, environment variables are 
  injected via `env:` in pod specs or from 
  Secrets and ConfigMaps

---

## Kubernetes connection

```yaml
# Kubernetes equivalent of Docker ENV:
containers:
  - name: sample-app
    image: sample-image
    env:
      - name: key1
        value: value1           # direct value (like ENV)
      - name: DB_PASSWORD
        valueFrom:
          secretKeyRef:         # from Kubernetes Secret
            name: db-secret
            key: password
```

---

## New commands learned today

| Command | Purpose |
|---------|---------|
| `ENV <key>=<value>` | Set environment variable in Dockerfile |
| `docker run -e <key>=<value>` | Override or add env var at runtime |
| `docker exec <c> env` | List all env vars inside container |
| `docker exec <c> printenv <key>` | Print specific env var value |
| `docker inspect <c> \| grep Env` | Inspect env vars from host |
| `cat > file << 'EOF'` | Create file with heredoc syntax |

---
