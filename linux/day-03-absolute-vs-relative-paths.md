# Day 08 — Linux: absolute vs relative paths

**Platform:** KillerCoda  
**Date:** April 2026  
**Topic:** Linux fundamentals — navigating the filesystem with paths

---

## What are paths in Linux?

A path is the address of a file or directory in 
the Linux filesystem. Understanding the difference 
between absolute and relative paths is one of the 
most fundamental Linux concepts — you will use 
this knowledge every single day in DevOps.

---

## Commands practised

| Command | What it does |
|---------|-------------|
| `mkdir root` | Creates a directory named `root` inside current location |
| `cd root` | Navigate INTO the `root` directory (relative path) |
| `cd /root` | Navigate to the `/root` directory (absolute path) |
| `cd ..` | Go up one level to parent directory |
| `cd ../../../var/log/nginx` | Navigate using relative path (multiple levels up then down) |
| `cd /var/log/nginx` | Navigate using absolute path |
| `pwd` | Print current working directory — shows exactly where you are |

---

## The Linux filesystem structure

```
/                        ← root of the entire filesystem
├── root/                ← home directory of the root USER
├── home/
│   ├── user1/
│   └── user2/
│       └── dir1/        ← example location
├── var/
│   └── log/
│       └── nginx/       ← example destination
├── etc/                 ← system configuration files
├── usr/
└── tmp/
```

`/` is the single starting point of everything 
in Linux. There is no `C:\` or `D:\` like Windows — 
everything lives under one root `/`.

---

## Absolute path vs relative path

### Absolute path
- Always starts with `/`
- References from the ROOT of the filesystem
- Works from ANY location in the system
- Like a full home address — always unambiguous

```bash
cd /var/log/nginx
# Works no matter where you currently are
```

### Relative path
- Does NOT start with `/`
- References from YOUR CURRENT location
- Changes meaning depending on where you are
- Like saying "turn left at the next corner" — 
  only makes sense if you know your starting point

```bash
cd ../../../var/log/nginx
# Only works if you are exactly 3 levels deep
# from the destination's branch
```

---

## The confusing but important example from this lesson

```bash
# Scenario: logged in as root user
# Current location: /root

mkdir root          # creates /root/root (a folder named root)
cd root             # now you are at /root/root (relative path)
pwd                 # prints: /root/root

cd ..               # back to /root
cd /root            # also takes you to /root (absolute path)
pwd                 # prints: /root
```

### Why is this confusing?

| Term | What it means |
|------|--------------|
| `/` | The filesystem root — the top of everything |
| `/root` | Home directory of the `root` USER |
| `root` (no slash) | A folder you created named "root" inside current dir |
| `root` user | The most powerful Linux user (like Administrator in Windows) |

```bash
cd /root    # goes to HOME directory of root user — ABSOLUTE
cd root     # goes into a folder named "root" in current dir — RELATIVE
```

These look almost identical but take you to 
completely different places.

---

## Navigating between distant directories

### Situation: you are in `/home/user2/dir1`
### Goal: reach `/var/log/nginx`

```bash
# Option 1 — relative path
cd ../../../var/log/nginx
# ../        = up to /home/user2
# ../../     = up to /home
# ../../../  = up to /
# then down into var/log/nginx

# Option 2 — absolute path (simpler and safer)
cd /var/log/nginx
```

### When to use which?

| Situation | Use |
|-----------|-----|
| Writing shell scripts | Absolute — scripts run from unpredictable locations |
| Quick terminal navigation | Relative — faster to type |
| Dockerfiles and CI/CD configs | Absolute — never assume a starting location |
| Navigating up one or two levels | Relative — `cd ../..` is quick |

---

## Key takeaways for DevOps

- Always use **absolute paths in scripts, Dockerfiles, 
  and CI/CD pipelines** — relative paths break when 
  the script is run from a different location
- `pwd` is your best friend when you are lost — 
  run it to confirm exactly where you are before 
  executing any destructive command
- The `/root` vs `root` distinction trips up even 
  experienced engineers — always double-check with 
  `pwd` after navigating
- Understanding `/` as the filesystem root is 
  essential for reading Docker volume mounts, 
  Kubernetes paths, and server configs

---

## Real DevOps example

```bash
# In a Dockerfile — always use absolute paths
WORKDIR /app              # absolute — safe
COPY ./config /app/config # clear source and destination

# In a shell script — absolute paths prevent bugs
LOG_DIR="/var/log/myapp"  # absolute path stored in variable
mkdir -p "$LOG_DIR"
cd "$LOG_DIR"
```

---

## New commands learned today

| Command | Purpose |
|---------|---------|
| `pwd` | Print current working directory |
| `cd /path` | Navigate using absolute path |
| `cd path` | Navigate using relative path |
| `cd ..` | Go up one directory level |
| `cd ../..` | Go up two directory levels |

---
