# Day 07 — Linux: mkdir command (Create directories)

**Platform:** KillerCoda  
**Date:** April 2026  
**Topic:** Linux fundamentals — creating directories and folder structures

---

## What is `mkdir`?

`mkdir` stands for **make directory**. It is the 
primary command for creating folders in Linux. 
In DevOps, you constantly create directory structures 
for projects, configs, logs, and deployments.

---

## Commands practised

| Command | What it does |
|---------|-------------|
| `ls -l` | List current directory contents in detail |
| `mkdir myfirstdirectory` | Create a single directory |
| `mkdir testdir{1..10}` | Create 10 directories in one command (testdir1 to testdir10) |
| `mkdir mydirectory anotherdirectory thirddirectory` | Create multiple differently named directories at once |
| `mkdir -p parentdir/childdir{01..100}` | Create parent + 100 child directories in one command |
| `ls -l parentdir` | List contents of a specific directory |

---

## Terminal observations

```bash
$ ls -l
# Shows current directory contents before we start

$ mkdir myfirstdirectory
$ ls -l
# myfirstdirectory now appears

$ mkdir testdir{1..10}
$ ls -l
# Creates: testdir1, testdir2, testdir3 ... testdir10
# All created with a single command

$ mkdir mydirectory anotherdirectory thirddirectory
$ ls -l
# All three directories created at once

$ mkdir -p parentdir/childdir{01..100}
$ ls -l parentdir
# Creates parentdir automatically, then inside it:
# childdir001, childdir002 ... childdir100
# Notice: we typed {01..10} but system auto-formatted
# to 3 digits (001, 002...) to match childdir100
```

---

## Key concept — brace expansion `{}`

Brace expansion is one of the most powerful 
shortcuts in Linux shell scripting:

```bash
mkdir testdir{1..10}
# Expands to: mkdir testdir1 testdir2 testdir3 ... testdir10

mkdir childdir{01..100}
# Expands to: mkdir childdir001 childdir002 ... childdir100

# You can also use letters:
mkdir dir{a..z}
# Creates: dira, dirb, dirc ... dirz
```

---

## Key concept — `-p` flag (parent directories)

Without `-p`, creating a nested directory fails 
if the parent does not exist:

```bash
# WITHOUT -p (fails if parentdir does not exist)
mkdir parentdir/childdir
# Error: cannot create directory: No such file or directory

# WITH -p (creates entire path in one go)
mkdir -p parentdir/childdir
# Creates parentdir first, then childdir inside it
# No error even if parentdir already exists
```

`-p` is the safe, reliable way to create 
directory structures in scripts and pipelines.

---

## Listing a specific directory

```bash
ls -l parentdir
# Lists contents of parentdir without navigating into it
# Same pattern works for any path:
ls -l /etc/
ls -l /var/log/
```

---

## Key takeaways for DevOps

- `mkdir -p` is used constantly in DevOps scripts — 
  always use `-p` when creating nested paths in 
  automation to avoid errors
- Brace expansion `{1..10}` is a shell feature that 
  works across many commands, not just `mkdir`
- Creating structured directories is the first step 
  in organising deployment configs, log directories, 
  and application folder structures on servers
- `ls -l <directory>` lets you inspect any folder 
  without navigating into it — useful when checking 
  configs in `/etc/` or logs in `/var/log/`

---

## Real DevOps example

```bash
# Creating a typical project structure in one command:
mkdir -p myapp/{config,logs,scripts,deployments}

# Result:
# myapp/
# ├── config/
# ├── logs/
# ├── scripts/
# └── deployments/
```

This pattern is commonly used in shell scripts 
that set up application environments automatically.

---

## New commands learned today

| Command | Purpose |
|---------|---------|
| `mkdir <name>` | Create a single directory |
| `mkdir -p <path>` | Create full nested path including parents |
| `mkdir dir{1..N}` | Create N directories using brace expansion |
| `ls -l <directory>` | List contents of a specific directory |

---

## What's next
- `rmdir` — remove empty directories
- `rm -rf` — remove directories with contents (use carefully)
- `cd` — navigate into directories
- `tree` — visualise full directory structure
