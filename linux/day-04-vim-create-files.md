# Day 04 — Linux: creating files with vim

**Platform:** KillerCoda  
**Date:** April 2026  
**Topic:** Linux fundamentals — introduction to vim editor

---

## What is vim?

`vim` stands for **Vi IMproved**. It is a powerful 
terminal-based text editor available on virtually 
every Linux system. Unlike `touch` which creates 
empty files, `vim` creates files WITH content.

In DevOps, vim is essential because:
- Remote servers often have NO GUI or graphical editor
- SSH sessions only give you a terminal
- vim is available on almost every Linux distro 
  by default — even minimal server installations
- Editing Dockerfiles, configs, and scripts on 
  servers requires a terminal editor

---

## vim vs touch — key difference

| Command | Creates file | Adds content | File size |
|---------|-------------|-------------|-----------|
| `touch myfile` | Yes | No | 0 bytes (empty) |
| `vim myfile` | Yes | Yes | Size of your content |

---

## Commands practised

| Command | What it does |
|---------|-------------|
| `vim mynewfile` | Open vim and create/edit a file named mynewfile |
| `i` | Enter INSERT mode — now you can type |
| `ESC` | Exit INSERT mode — return to COMMAND mode |
| `:wq` | Write (save) and quit vim |
| `ls -l mynewfile` | Confirm file exists and has non-zero size |

---

## vim has TWO modes — this is the key concept

Most beginners struggle with vim because it has 
modes. Understanding modes is everything:

```
COMMAND mode (default when you open vim)
│
│  Press i
▼
INSERT mode (you can now type text)
│
│  Press ESC
▼
COMMAND mode (back to commands)
│
│  Type :wq and press Enter
▼
Saved and exited vim
```

| Mode | What you can do | How to enter |
|------|----------------|-------------|
| COMMAND mode | Run vim commands, navigate | Press `ESC` |
| INSERT mode | Type and edit text | Press `i` |

---

## Step by step — what was done in this lesson

```bash
# Step 1 — open vim and create the file
vim mynewfile
# vim opens, you are in COMMAND mode
# screen shows empty buffer with ~ on each line

# Step 2 — enter INSERT mode
i
# Bottom of screen now shows: -- INSERT --

# Step 3 — type your content
this is my great line

# Step 4 — exit INSERT mode
ESC
# -- INSERT -- disappears from bottom

# Step 5 — save and quit
:wq
# : opens the command prompt at bottom
# w = write (save)
# q = quit

# Step 6 — confirm the file was created with content
ls -l mynewfile
# Output: -rw-r--r-- 1 root root 22 Apr 2026 mynewfile
#                              ^ NOT 0 — file has content!
```

---

## Essential vim commands reference

### Exiting vim (the most Googled thing in Linux)

| Command | What it does |
|---------|-------------|
| `:wq` | Save and quit |
| `:q` | Quit (only works if no unsaved changes) |
| `:q!` | Force quit WITHOUT saving |
| `:w` | Save only, stay in vim |
| `ZZ` | Save and quit (shortcut, no colon needed) |

### Navigation in COMMAND mode

| Key | Action |
|-----|--------|
| `h` | Move left |
| `l` | Move right |
| `j` | Move down |
| `k` | Move up |
| `gg` | Jump to first line |
| `G` | Jump to last line |
| `0` | Jump to start of line |
| `$` | Jump to end of line |

### Editing in COMMAND mode

| Key | Action |
|-----|--------|
| `i` | Insert before cursor |
| `a` | Insert after cursor |
| `o` | Open new line below and insert |
| `dd` | Delete entire current line |
| `yy` | Copy (yank) current line |
| `p` | Paste below current line |
| `u` | Undo last action |

---

## Key takeaways for DevOps

- vim is the most important terminal editor 
  to learn — it is available everywhere
- The number one reason people get "stuck" in vim 
  is forgetting which mode they are in — always 
  check the bottom of the screen for `-- INSERT --`
- `:q!` is your emergency exit — use it when 
  something goes wrong and you want to quit 
  without saving changes
- You will use vim to edit Dockerfiles, 
  Kubernetes YAML configs, nginx configs, 
  shell scripts, and `.env` files on servers
- `nano` is easier for beginners but vim is 
  faster once learned — invest the time now

---

## Real DevOps usage examples

```bash
# Edit a Dockerfile on a remote server
vim Dockerfile

# Edit nginx configuration
vim /etc/nginx/nginx.conf

# Edit Kubernetes deployment YAML
vim deployment.yaml

# Edit environment variables file
vim .env

# Quick edit a shell script
vim deploy.sh
```

---

## New commands learned today

| Command | Purpose |
|---------|---------|
| `vim <filename>` | Open or create a file in vim |
| `i` | Enter INSERT mode to start typing |
| `ESC` | Return to COMMAND mode |
| `:wq` | Save file and exit vim |
| `:q!` | Exit vim WITHOUT saving |

---
