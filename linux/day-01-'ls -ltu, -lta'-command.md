# Day 01 — Linux: ls sorting options

**Platform:** KillerCoda  
**Date:** April 2026  
**Topic:** Linux fundamentals — sorting files and understanding timestamps

---

## What is sorting in `ls`?

By default, `ls` sorts files alphabetically. Linux 
provides additional sorting options based on file 
timestamps, which are critical in DevOps for 
debugging, log analysis, and auditing file changes.

---

## Understanding Linux timestamps

This is one of the most important concepts in Linux 
file management. Every file has THREE timestamps:

| Timestamp | Full name | What triggers it |
|-----------|-----------|-----------------|
| `atime`   | Access time | File was read or opened |
| `mtime`   | Modification time | File **content** was changed |
| `ctime`   | Change time | File **metadata** changed (permissions, location) |

### What is a timestamp?
A timestamp is a numerical representation of time — 
specifically the number of seconds passed since 
**Unix epoch** (midnight, 1st January 1970).
```bash
$ date +%s
1635797690        ← this is the Unix timestamp (seconds)

$ date
Mon 01 Nov 2021 08:14:52 PM UTC   ← human-readable form
```

---

## Commands practised

| Command | What it does |
|---------|-------------|
| `ls -lt`  | Sort by `mtime` — newest modified file first |
| `ls -ltu` | Sort by `atime` — newest accessed file first |
| `ls -ltc` | Sort by `ctime` — newest metadata change first |
| `date`    | Shows current date and time |
| `touch theNewestFile` | Creates a new empty file (updates `atime` + `mtime`) |
| `echo "hello world!" > file-02` | Writes content into file (updates `mtime`) |
| `chmod 444 file-01` | Changes file permissions (updates `ctime` only) |

---

## Key experiment performed

This experiment proves the difference between 
`mtime` and `ctime` in a real terminal:
```bash
# Step 1 — create a new file
touch theNewestFile
ls -ltu    # theNewestFile appears at top (atime updated)
ls -ltc    # theNewestFile appears at top (ctime updated)

# Step 2 — add content to a file
echo "hello world!" > file-02
ls -ltu    # file-02 moves to top (content accessed)
ls -ltc    # file-02 moves to top (metadata also changed)

# Step 3 — change permissions only
chmod 444 file-01
ls -ltu    # file-01 does NOT move (content not touched)
ls -ltc    # file-01 moves to top (metadata changed!)
```

### What this proves
- `chmod` updates `ctime` but NOT `mtime` or `atime`
- `echo > file` updates `mtime` (content changed)
- `touch` updates all timestamps on a new file
- This distinction matters when debugging — 
  a file can appear "recently changed" in `ctime` 
  even if its content was never touched

---

## Key takeaways for DevOps

- Use `ls -lt` daily to find the most recently 
  modified files on a server — essential for 
  debugging deployments and checking log updates
- `ls -ltu` helps identify which files were recently 
  accessed — useful in security audits
- `ls -ltc` reveals permission or ownership changes — 
  critical for tracking unauthorised access
- `chmod 444` makes a file read-only for everyone — 
  a common pattern for protecting config files 
  in production environments
- Understanding `atime` vs `mtime` vs `ctime` is 
  frequently asked in DevOps and Linux admin interviews

---

## New commands learned today

| Command | Purpose |
|---------|---------|
| `date` | Display current date and time |
| `date +%s` | Display current Unix timestamp in seconds |
| `touch filename` | Create a new empty file |
| `echo "text" > filename` | Write text into a file |
| `chmod 444 filename` | Set file to read-only for all users |

---
