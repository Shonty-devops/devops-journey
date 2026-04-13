# Day 02 — Linux: man helper arguments (whatis, -f, -k, -w)

**Platform:** KillerCoda  
**Date:** April 2026  
**Topic:** Linux fundamentals — quick lookup commands for man pages

---

## What are man helper arguments?

Instead of opening a full man page, Linux provides 
quick lookup commands that give you fast answers 
without reading an entire manual. These are 
extremely useful on production servers where 
you need information quickly.

---

## Commands practised

| Command | What it does |
|---------|-------------|
| `whatis ls` | Prints a one-line description of `ls` |
| `man -f ls` | Identical to `whatis` — shows short description |
| `man -k ls` | Searches ALL man pages for keyword `ls` and lists matches |
| `man -w ls` | Shows the file path where the man page is stored |

---

## Terminal observations

```bash
$ whatis ls
ls (1) - list directory contents

$ man -f ls
ls (1) - list directory contents
# Identical output to whatis

$ man -k ls
# Returns ALL man pages containing "ls" keyword:
# ls (1)         - list directory contents
# lsblk (8)      - list block devices
# lscpu (1)      - display CPU information
# lsmod (8)      - show loaded kernel modules
# lsof (8)       - list open files
# lspci (8)      - list PCI devices
# lsusb (1)      - list USB devices
# ... and many more

$ man -w ls
/usr/share/man/man1/ls.1.gz
# This is where the actual man page file lives on disk
```

---

## Key difference — `-f` vs `-k`

```bash
man -f ls    # exact match only — finds pages named "ls"
man -k ls    # keyword search — finds ALL pages mentioning "ls"
```

Think of it this way:
- `-f` is like searching a book index for exact title
- `-k` is like using Ctrl+F to find every mention 
  of a word throughout the entire book

---

## Why `man -k` is powerful in DevOps

If you forget a command name but remember what 
it does, `-k` helps you find it:

```bash
man -k "list"       # find all commands related to listing
man -k "network"    # find all commands related to networking
man -k "process"    # find all commands related to processes
man -k "disk"       # find all commands related to disk management
```

This is equivalent to the `apropos` command:
```bash
apropos ls          # same output as man -k ls
```

---

## Where are man pages stored?

```bash
$ man -w ls
/usr/share/man/man1/ls.1.gz
```

Breaking down this path:
| Part | Meaning |
|------|---------|
| `/usr/share/man/` | Root directory for all man pages |
| `man1/` | Section 1 folder (shell commands) |
| `ls.1.gz` | The man page file, compressed with gzip |

Man pages are stored as compressed `.gz` files 
to save disk space on servers.

---

## Key takeaways for DevOps

- `whatis <command>` is the fastest way to check 
  what an unfamiliar command does — use it before 
  opening the full man page
- `man -k <keyword>` is invaluable when you know 
  what you want to do but not which command does it
- `man -w <command>` is useful for understanding 
  Linux filesystem structure — man pages live in 
  `/usr/share/man/` on most distros
- These commands work offline on any Linux server — 
  no internet needed, no Googling required

---

## Quick reference card

```bash
whatis <cmd>     # one-line description — what is this?
man -f <cmd>     # same as whatis
man -k <keyword> # search all man pages — find the right command
man -w <cmd>     # where is this man page stored on disk?
apropos <keyword> # same as man -k
```

---

## New commands learned today

| Command | Purpose |
|---------|---------|
| `whatis` | One-line description of any command |
| `apropos` | Search man pages by keyword (same as `man -k`) |

---

## What's next
- `man -k network` — discover networking commands
- `man -k process` — discover process management commands
- `tldr <command>` — modern alternative to man pages 
  with practical examples (needs installation)
