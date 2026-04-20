# Day 03 — Linux: navigating back home

**Platform:** KillerCoda  
**Date:** April 2026  
**Topic:** Linux fundamentals — four ways to return to home directory

---

## What is the home directory?

Every user in Linux has a dedicated home directory 
where their personal files, configs, and settings live.

| User | Home directory |
|------|---------------|
| `root` user | `/root` |
| Regular user | `/home/username` |

No matter how deep you navigate into the filesystem, 
you always need a quick way back home.

---

## Commands practised

| Command | What it does |
|---------|-------------|
| `cd /root` | Navigate home using absolute path (manual) |
| `cd $HOME` | Navigate home using the built-in HOME variable |
| `echo $HOME` | Display the value stored in the HOME variable |
| `cd ~` | Navigate home using tilde shortcut |
| `cd` | Navigate home — shortest version, no arguments needed |
| `pwd` | Confirm current location after each navigation |

---

## Four ways to go home — from longest to shortest

```bash
# Starting point — navigate somewhere deep first
cd /var/log
pwd
# Output: /var/log

# Method 1 — absolute path (manual, but you must know the path)
cd /root
pwd
# Output: /root

# Method 2 — $HOME variable
cd /var/log
cd $HOME
pwd
# Output: /root

# Method 3 — tilde shortcut
cd /var/log
cd ~
pwd
# Output: /root

# Method 4 — bare cd (shortest)
cd /var/log
cd
pwd
# Output: /root
```

---

## The `$HOME` variable

`$HOME` is a built-in Linux environment variable 
that always holds the path to the current user's 
home directory.

```bash
echo $HOME
# Output for root user:    /root
# Output for regular user: /home/ayush
```

Variables in Linux are referenced with `$` prefix. 
`$HOME` is one of several important built-in 
environment variables you will use in DevOps:

| Variable | Contains |
|----------|---------|
| `$HOME` | Current user's home directory path |
| `$USER` | Current logged-in username |
| `$PATH` | Directories where Linux looks for commands |
| `$PWD` | Current working directory (same as `pwd`) |
| `$SHELL` | Path to the current shell being used |

---

## The tilde `~` shortcut

`~` is simply a shorthand alias for `$HOME`. 
It is recognised by the shell and expanded 
automatically:

```bash
cd ~              # goes to /root (or /home/username)
cd ~/Documents    # goes to /root/Documents
cat ~/.bashrc     # reads the .bashrc file in home directory
```

The `~` is extremely common in Linux — you will 
see it constantly in documentation, scripts, 
and config files.

---

## Comparison table

| Method | Command | Length | Best used when |
|--------|---------|--------|---------------|
| Absolute path | `cd /root` | Long | You need to be explicit in scripts |
| HOME variable | `cd $HOME` | Medium | Scripts that work for any user |
| Tilde | `cd ~` | Short | Quick terminal navigation |
| Bare cd | `cd` | Shortest | Quick terminal navigation |

---

## Key takeaways for DevOps

- Use `cd` or `cd ~` for quick navigation in terminal
- Use `$HOME` in shell scripts — it works for ANY 
  user, not just root, making scripts portable
- Never hardcode `/root` in scripts — if the script 
  runs as a different user, the path breaks. 
  Use `$HOME` instead
- `~` is widely used in config file paths — 
  e.g. `~/.ssh/config`, `~/.bashrc`, `~/.kube/config`
- `~/.kube/config` is particularly important — 
  this is where Kubernetes stores cluster credentials

---

## Real DevOps example

```bash
# WRONG — hardcoded path, breaks for other users
cp config.yaml /root/.kube/config

# RIGHT — portable, works for any user
cp config.yaml $HOME/.kube/config

# Common paths you will use as a DevOps engineer
~/.ssh/           # SSH keys and config
~/.kube/config    # Kubernetes cluster credentials
~/.bashrc         # Shell configuration and aliases
~/.gitconfig      # Git user settings
```

---

## New commands learned today

| Command | Purpose |
|---------|---------|
| `echo $VARIABLE` | Display the value of any environment variable |
| `cd $HOME` | Go home using HOME variable |
| `cd ~` | Go home using tilde shortcut |
| `cd` | Go home — no arguments needed |

---

## What's next
- `echo $PATH` — understand how Linux finds commands
- `export VARIABLE=value` — set your own environment variables
- `~/.bashrc` — customise your shell environment
- `env` — display all environment variables at once
