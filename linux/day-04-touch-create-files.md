# Day 04 — Linux: touch command and wildcards

**Platform:** KillerCoda  
**Date:** April 2026  
**Topic:** Linux fundamentals — creating files and using wildcard characters

---

## What is the `touch` command?

`touch` creates new empty files instantly. 
It also updates the timestamp of existing files 
without changing their content (which is why 
we saw it in the timestamps lesson — Day 03).

In DevOps, `touch` is used to:
- Create placeholder files in scripts
- Create lock files to prevent duplicate processes
- Update timestamps on config files to trigger 
  reloads without editing content

---

## What are wildcards?

Wildcards are special characters that match 
patterns in filenames. They allow you to work 
with multiple files at once without typing 
each name individually.

| Wildcard | Meaning | Example |
|----------|---------|---------|
| `*` | Any string of characters (including empty) | `ls my*file` matches myfile, my01file, myABCfile |
| `?` | Any single character (exactly one) | `ls try?` matches try1, try2 but NOT try01 |

---

## Commands practised

| Command | What it does |
|---------|-------------|
| `touch testfile` | Create a single empty file |
| `ls -l testfile` | Confirm file exists and check its size (0 bytes) |
| `touch my{01..100}file` | Create 100 files using brace expansion |
| `ls my*file` | List all files matching pattern `my + anything + file` |
| `touch try1 try2 try01` | Create three specific files |
| `ls try*` | Lists try1, try2, AND try01 (too broad) |
| `ls try?` | Lists ONLY try1 and try2 (exactly one char after try) |
| `mkdir testdir` | Create a test directory |
| `touch testdir/mytest{01..1000} testdir/file{01..1000}` | Create 2000 files inside testdir |

---

## Terminal observations

```bash
