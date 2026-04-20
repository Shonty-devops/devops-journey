# Day 03 — Linux: deleting directories (rmdir and rm)

**Platform:** KillerCoda  
**Date:** April 2026  
**Topic:** Linux fundamentals — safely removing directories

---

## What commands delete directories?

Linux has two commands for deletion:

| Command | Works on | Condition |
|---------|----------|-----------|
| `rmdir` | Directories only | Directory must be EMPTY |
| `rm` | Files and directories | With `-rf` flags, removes anything |

---

## Commands practised

| Command | What it does |
|---------|-------------|
| `rmdir root` | Remove single empty directory named `root` |
| `rmdir testdir{1..10}` | Remove 10 directories using brace expansion |
| `rmdir parentdir` | Fails — directory is not empty |
| `rmdir -p maindir/childdir` | Remove childdir then maindir if it becomes empty |
| `rmdir parentdir/*` | Remove all contents of parentdir first |
| `rmdir parentdir` | Then remove the now-empty parentdir |
| `rm parentdir` | Fails — rm alone does not work on directories |
| `rm -rf anotherparentdir` | Forcefully remove directory and ALL contents recursively |
| `rm -rf /` | Attempts to delete the ENTIRE filesystem — extremely dangerous |

---

## Terminal observations

```bash
