# Day 02 — Linux: man sections (Manual page sections)

**Platform:** KillerCoda  
**Date:** April 2026  
**Topic:** Linux fundamentals — navigating man page sections

---

## What are man page sections?

The Linux manual is divided into 9 sections. Each 
section covers a different category of documentation. 
Knowing sections helps you find the RIGHT manual page 
when a command exists in multiple contexts.

---

## The 9 sections of the Linux manual

| Section | Category | Example |
|---------|----------|---------|
| `1` | Executable programs / shell commands | `ls`, `cd`, `grep` |
| `2` | System calls (kernel functions) | `fork()`, `read()` |
| `3` | Library calls (program libraries) | `printf()`, `malloc()` |
| `4` | Special files | `/dev/null`, `/dev/sda` |
| `5` | File formats and conventions | `/etc/passwd`, `/etc/hosts` |
| `6` | Games | — |
| `7` | Miscellaneous / macro packages | `man(7)`, `groff(7)` |
| `8` | System administration commands (root) | `iptables`, `mount` |
| `9` | Kernel routines (non-standard) | Kernel internals |

---

## Commands practised

| Command | What it does |
|---------|-------------|
| `man 8 ls` | Tries to open section 8 of ls manual — does not exist |
| `man -f ls` | Shows ALL available sections for the `ls` command |
| `man -f intro` | Shows all sections available for `intro` |
| `man intro` | Opens default (section 1) intro page |
| `man 1 intro` | Explicitly opens section 1 of intro |
| `man 8 intro` | Opens section 8 of intro (system admin intro) |
| `q` | Quit any man page |

---

## Terminal observations

```bash
$ man 8 ls
# Output: No manual entry for ls in section 8
# ls is a user command — it only exists in section 1

$ man -f ls
# Output: ls (1) - list directory contents
# Confirms ls only has 1 section available

$ man -f intro
# Output shows multiple sections:
# intro (1) - introduction to user commands
# intro (2) - introduction to system calls
# intro (3) - introduction to library functions
# intro (8) - introduction to admin commands
```

---

## Key concept — why sections matter

Some command names exist in MULTIPLE sections 
with completely different meanings:

```bash
man printf        # section 1 — the shell command
man 3 printf      # section 3 — the C library function

man passwd        # section 1 — the passwd command
man 5 passwd      # section 5 — the /etc/passwd file format
```

Without specifying a section, `man` always opens 
the lowest-numbered section by default.

---

## Key takeaways for DevOps

- Section `1` is what you use daily — shell commands
- Section `5` is critical for DevOps — config file 
  formats like `/etc/passwd`, `/etc/fstab`, `/etc/hosts`
- Section `8` covers admin commands like `iptables`, 
  `mount`, `systemctl` — things you run as root on servers
- `man -f <command>` before opening a man page tells 
  you exactly which sections exist — saves time
- In interviews: knowing that `man passwd` and 
  `man 5 passwd` show different things demonstrates 
  genuine Linux depth

---

## Quick reference — most useful sections for DevOps

| Section | Why DevOps engineers use it |
|---------|-----------------------------|
| `1` | Daily commands — ls, grep, curl, ssh |
| `5` | Config file formats — nginx.conf, fstab, passwd |
| `8` | Admin tools — iptables, mount, useradd, systemctl |

---

## What's next
- `man 5 passwd` — understand the passwd file format
- `man 8 iptables` — firewall rules reference
- `apropos <keyword>` — search man pages by keyword 
  when you do not know the command name
