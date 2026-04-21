# Day 03 — Docker: using ENTRYPOINT and CMD together

**Platform:** KillerCoda  
**Date:** April 2026  
**Topic:** Docker fundamentals — combining ENTRYPOINT and CMD

---

## Task summary

- Use BOTH CMD and ENTRYPOINT together in Dockerfile
- Build image named `image-echo`
- Inspect image to verify both CMD and ENTRYPOINT
- Run container four different ways via CLI

---

## The Dockerfile

```dockerfile
FROM alpine
ENTRYPOINT ["echo"]
CMD ["hi, from container!"]
```

ENTRYPOINT sets the fixed executable (`echo`)
CMD sets the default argument (`hi, from container!`)
Together they run: `echo "hi, from container!"`

---

## Full solution — step by step

```bash
# Step 1 — edit the Dockerfile
vim /root/Dockerfile
# Add both lines as shown above
# Save: ESC → :wq

# Step 2 — build the image
docker build -t image-echo /root

# Step 3 — inspect CMD and ENTRYPOINT
docker inspect image-echo | grep -A 3 "Cmd\|Entrypoint"
# Expected output:
# "Cmd": [
#     "hi, from container!"
# ],
# "Entrypoint": [
#     "echo"
# ]

# Step 4 — run with default CMD and ENTRYPOINT
docker run image-echo
# Runs: echo hi, from container!
# Output: hi, from container!

# Step 5 — run with another message via CLI
docker run image-echo "hi, from the updated image"
# Runs: echo "hi, from the updated image"
# Output: hi, from the updated image
# CLI argument REPLACES CMD, ENTRYPOINT stays as echo

# Step 6 — set CMD to date via CLI
docker run image-echo date
# Runs: echo date
# Output: date
# "date" replaces CMD as argument passed to echo

# Step 7 — set ENTRYPOINT to date via CLI
docker run --entrypoint date image-echo
# Runs: date (ignores CMD completely)
# Output: Mon Apr 20 10:00:00 UTC 2026
```

---

## What each run command produced

| Command | ENTRYPOINT used | CMD used | Output |
|---------|----------------|----------|--------|
| `docker run image-echo` | `echo` | `hi, from container!` | hi, from container! |
| `docker run image-echo "hi, from the updated image"` | `echo` | `hi, from the updated image` | hi, from the updated image |
| `docker run image-echo date` | `echo` | `date` | date (printed as text) |
| `docker run --entrypoint date image-echo` | `date` | ignored | current date/time |

---

## Visual model — how they combine

```
Dockerfile defines:
┌─────────────────┐     ┌──────────────────────┐
│  ENTRYPOINT     │  +  │       CMD            │
│  ["echo"]       │     │  ["hi, from          │
│                 │     │   container!"]        │
└─────────────────┘     └──────────────────────┘
         │                        │
         ▼                        ▼
    fixed executable         default argument
    always runs              easily replaced

Result: echo "hi, from container!"

When you run: docker run image-echo "new message"
┌─────────────────┐     ┌──────────────────────┐
│  ENTRYPOINT     │  +  │    CLI argument      │
│  ["echo"]       │     │  ["new message"]     │
└─────────────────┘     └──────────────────────┘
Result: echo "new message"

When you run: docker run --entrypoint date image-echo
┌─────────────────┐     ┌──────────────────────┐
│  CLI entrypoint │  +  │    CMD ignored       │
│  [date]         │     │    completely        │
└─────────────────┘     └──────────────────────┘
Result: date
```

---

## The four scenarios — summary rules

```bash
# Scenario 1 — nothing overridden
docker run image-echo
→ ENTRYPOINT + CMD = echo "hi, from container!"

# Scenario 2 — message overridden via CLI
docker run image-echo "hi, from the updated image"
→ ENTRYPOINT + CLI arg = echo "hi, from the updated image"
  (CMD is replaced by CLI argument)

# Scenario 3 — CMD set to date via CLI
docker run image-echo date
→ ENTRYPOINT + CLI arg = echo date
  (date is passed as text argument to echo)

# Scenario 4 — ENTRYPOINT overridden via CLI
docker run --entrypoint date image-echo
→ date runs, CMD completely ignored
```

---

## Key concept — override priority

```
What runs = ENTRYPOINT + CMD

Override rules:
CLI argument after image  →  replaces CMD only
--entrypoint flag         →  replaces ENTRYPOINT
                             (CMD also ignored)
```

---

## Key takeaways for DevOps

- Using ENTRYPOINT + CMD together is the 
  most professional and flexible Dockerfile pattern
- ENTRYPOINT = the tool (`echo`, `python`, `nginx`)
- CMD = the default config/argument for that tool
- This pattern makes containers behave like 
  executable commands — very clean for tooling images
- Real world example:

```dockerfile
# A database backup tool container
FROM alpine
ENTRYPOINT ["pg_dump"]
CMD ["--help"]

# Default run shows help:
docker run db-backup
# → pg_dump --help

# Actual backup with args:
docker run db-backup -h localhost -U postgres mydb
# → pg_dump -h localhost -U postgres mydb
```

---

## New commands learned today

| Command | Purpose |
|---------|---------|
| `docker inspect <img> \| grep -A 3 "Cmd\|Entrypoint"` | Check both CMD and ENTRYPOINT |
| `docker run <image> "custom argument"` | Replace CMD with custom argument |
| `docker run --entrypoint <cmd> <image>` | Replace ENTRYPOINT entirely |

---
- `ARG` — pass build-time variables
- `VOLUME` — persist data outside the container
- `EXPOSE` — document which ports the container uses
