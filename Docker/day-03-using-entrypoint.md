# Day 03 — Docker: using ENTRYPOINT in practice

**Platform:** KillerCoda  
**Date:** April 2026  
**Topic:** Docker fundamentals — hands-on ENTRYPOINT task

---

## Task summary

- Remove `CMD` from existing Dockerfile
- Add `ENTRYPOINT` to run `echo hi, from container!`
- Build image named `entrypoint-echo`
- Inspect the image to verify ENTRYPOINT
- Run container with ENTRYPOINT overridden via CLI
- Run container with CMD set via CLI

---

## The Dockerfile — before and after

```dockerfile
# BEFORE
FROM alpine
CMD ["echo", "hi, from container!"]

# AFTER
FROM alpine
ENTRYPOINT ["echo", "hi, from container!"]
```

---

## Full solution — step by step

```bash
# Step 1 — open and edit the Dockerfile
vim /root/Dockerfile
# Remove CMD line
# Add: ENTRYPOINT ["echo", "hi, from container!"]
# Save: ESC → :wq

# Step 2 — build the image
docker build -t entrypoint-echo /root

# Step 3 — inspect to verify ENTRYPOINT
docker inspect entrypoint-echo | grep -A 3 "Entrypoint"
# Expected output:
# "Entrypoint": [
#     "echo",
#     "hi, from container!"
# ]

# Step 4 — run normally
docker run entrypoint-echo
# Output: hi, from container!

# Step 5 — overwrite ENTRYPOINT with date via CLI
docker run --entrypoint date entrypoint-echo
# Output: Mon Apr 20 10:00:00 UTC 2026
# date command runs INSTEAD of echo

# Step 6 — set CMD to date via CLI
docker run entrypoint-echo date
# Output: hi, from container! date
# "date" is passed as ARGUMENT to echo
```

---

## What each run command produced

| Command | What ran | Output |
|---------|---------|--------|
| `docker run entrypoint-echo` | `echo hi, from container!` | hi, from container! |
| `docker run --entrypoint date entrypoint-echo` | `date` | current date/time |
| `docker run entrypoint-echo date` | `echo date` | date (as text, passed to echo) |

---

## Key observations

### Overwriting ENTRYPOINT with `--entrypoint`
```bash
docker run --entrypoint date entrypoint-echo
# --entrypoint flag COMPLETELY replaces the entrypoint
# The original "echo hi, from container!" is ignored
# "date" command runs instead
```

### Setting CMD via CLI
```bash
docker run entrypoint-echo date
# "date" is appended as an ARGUMENT to ENTRYPOINT
# ENTRYPOINT "echo" remains unchanged
# Effectively runs: echo date
# Output is the word "date" printed as text
```

---

## Key takeaways for DevOps

- `--entrypoint` flag is extremely useful for 
  debugging containers — you can override any 
  entrypoint and drop into a shell:
```bash
docker run --entrypoint sh entrypoint-echo
# Opens a shell inside the container
# Even if the container was built to run something else
```
- Arguments passed after the image name in 
  `docker run` always go to ENTRYPOINT as CMD — 
  they do not replace ENTRYPOINT
- This behaviour is consistent across all Docker 
  containers — knowing it saves debugging time

---

## New commands learned today

| Command | Purpose |
|---------|---------|
| `vim /root/Dockerfile` | Edit Dockerfile at specific path |
| `docker build -t <name> <path>` | Build image from path |
| `docker inspect <image> \| grep -A 3 "Entrypoint"` | Check entrypoint of built image |
| `docker run --entrypoint <cmd> <image>` | Override ENTRYPOINT at runtime |
| `docker run <image> <arg>` | Pass argument to ENTRYPOINT at runtime |
