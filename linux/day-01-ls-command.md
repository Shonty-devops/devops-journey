# Day 01 — Linux: ls command (List Files)

**Platform:** KillerCoda  
**Date:** April 2026  
**Topic:** Linux fundamentals — listing directory contents

---

## What is `ls`?

`ls` is one of the core Linux/Unix commands used to list 
the contents of a directory. It is used across all Linux 
and Unix systems and is foundational to navigating any 
server or container environment in DevOps.

---

## Commands practised

| Command | Output behaviour |
|---|---|
| `ls` | Lists files and directories with colour coding |
| `ls --color=no` | Lists everything in plain white text, no colours |
| `ls --color=yes` | Forces colour output (directories in cyan, files in white) |

---

## What I observed in the terminal
```bash
root@ubuntu:~$ ls
File-01.txt  file-01  file-01.txt  file-02  filesystem  
notmyfile  notmyfile2  testDir  testdir

root@ubuntu:~$ ls --color=no
# Same output but all in plain white — no colour distinction

root@ubuntu:~$ ls --color=yes
# Directories appear in cyan/teal
# Regular files appear in white
```

## Key takeaways for DevOps

- Colour coding in `ls` helps instantly identify 
  **directories vs files vs executables** on a server
- In scripts and pipelines, always use `ls --color=no` 
  to avoid ANSI colour codes breaking your output parsing
- `filesystem` and `testDir` appearing in cyan confirms 
  they are directories, not files

---

## What's next
- `ls -la` — view hidden files + permissions
- `ls -lh` — human-readable file sizes  
- `ls -lt` — sort by last modified time
