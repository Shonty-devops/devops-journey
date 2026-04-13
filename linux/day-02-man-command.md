# Day 02 — Linux: man command (Manual pages)

**Platform:** KillerCoda  
**Date:** April 2026  
**Topic:** Linux fundamentals — using the manual to learn any command

---

## What is the `man` command?

`man` stands for **manual**. It is your built-in 
documentation system in Linux. Every command 
installed on a Linux system has a manual page 
that explains exactly how it works, what arguments 
it accepts, and what options are available.

In DevOps, `man` is critically important because:
- You will encounter commands you have never used before
- Server environments often have no internet access
- It is faster than Googling for quick syntax checks

---

## Commands practised

| Command | What it does |
|---------|-------------|
| `man man` | Opens the manual page for the `man` command itself |
| `man ls`  | Opens the full manual page for the `ls` command |
| `q`       | Quit/exit any man page |
| Arrow keys | Navigate up and down through the man page |

---

## How to navigate a man page

| Key | Action |
|-----|--------|
| `Arrow Up / Down` | Scroll line by line |
| `Space` | Scroll down one full page |
| `b` | Scroll back one full page |
| `G` | Jump to the end of the document |
| `g` | Jump to the beginning |
| `/keyword` | Search for a keyword inside the man page |
| `n` | Jump to next search result |
| `q` | Quit and return to terminal |

---

## Structure of a man page

Every man page follows the same structure:

| Section | What it contains |
|---------|-----------------|
| `NAME` | Command name and one-line description |
| `SYNOPSIS` | Syntax — how to use the command |
| `DESCRIPTION` | Full explanation of what the command does |
| `OPTIONS` | All available flags and arguments |
| `EXAMPLES` | Usage examples (not always present) |
| `SEE ALSO` | Related commands worth exploring |

---

## What `man ls` revealed

Running `man ls` shows every possible flag for the 
`ls` command in one place. This is how you discover 
flags beyond the basics:

```bash
man ls
# Inside the man page you can now see ALL ls options:
# -a, -A, -l, -h, -t, -u, -c, -R, -S and many more
# Everything we practised in Day 01-03 is documented here
```

---

## Key takeaways for DevOps

- Before Googling any command, try `man <command>` first — 
  it works offline on any Linux server
- In interviews, saying "I check the man page" shows 
  maturity and independence as an engineer
- `man` pages are the source of truth — blog posts 
  can be outdated, man pages are always current 
  for your installed version
- Most DevOps tools also support `--help` flag as 
  a quick alternative: `ls --help`, `docker --help`,
  `kubectl --help`

---

## Useful pattern to remember

```bash
# If you forget a flag, do this:
man ls | grep "human"    # search man page output for keyword
# or simply open it and type /human to search inside
```

---

## New commands learned today

| Command | Purpose |
|---------|---------|
| `man <command>` | Open full manual for any command |
| `man man` | Read the manual about the manual itself |
| `/keyword` | Search inside any man page |
