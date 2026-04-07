# Day 01 — Linux: ls flags (List all files including hidden)

**Platform:** KillerCoda  
**Date:** April 2026  
**Topic:** Linux fundamentals — listing hidden files and directories

---

## What are hidden files in Linux?

In Linux, files and directories that begin with a `.` 
(dot) are called **dotfiles**. These are hidden objects 
and are NOT shown by the standard `ls` command. 
They are present in almost every Linux directory and 
are commonly used for configuration files.

---

## Commands practised

| Command | What it does |
|---------|-------------|
| `ls`    | Lists only visible files — hidden files not shown |
| `ls -a` | Lists ALL files including hidden dotfiles |
| `ls -A` | Lists almost all — hidden files shown but excludes `.` and `..` |
| `ls -al`| Combines `-a` and `-l` — all files with full details |
| `ls .`  | Lists current directory (same as plain `ls`) |
| `ls ..` | Lists the parent directory contents |

---

## Key concepts learned

### The dot notation
| Symbol | Meaning |
|--------|---------|
| `.`    | Current directory |
| `..`   | Parent directory |

### Difference between `-a` and `-A`
- `-a` (lowercase) = shows absolutely everything, 
   including `.` and `..`
- `-A` (uppercase) = shows all hidden files but 
   skips `.` and `..` — cleaner output

---

## Terminal observations
root@ubuntu:~$ ls -a
.   .bash_history  .hidden-file  .ssh    .vimrc      File-01.txt  file-01.txt  filesystem  notmyfile2  testdir
..  .bashrc        .profile      .theia  .wget-hsts  file-01      file-02      notmyfile   testDir

root@ubuntu:~$ ls .
File-01.txt  file-01  file-01.txt  file-02  filesystem  notmyfile  notmyfile2  testDir  testdir

root@ubuntu:~$ ls ..
bin                boot  etc   lib                lib64       media  opt   root  sbin                snap  swapfile  tmp  var
bin.usr-is-merged  dev   home  lib.usr-is-merged  lost+found  mnt    proc  run   sbin.usr-is-merged  srv   sys       usr

root@ubuntu:~$ ls -A 
.bash_history  .hidden-file  .ssh    .vimrc      File-01.txt  file-01.txt  filesystem  notmyfile2  testdir
.bashrc        .profile      .theia  .wget-hsts  file-01      file-02      notmyfile   testDir

root@ubuntu:~$ ls -al 
total 568
drwx------  6 root      root        4096 Apr  7 10:30 .
drwxr-xr-x 22 root      root        4096 Apr  1 08:11 ..
-rw-------  1 root      root          10 Feb 10  2025 .bash_history
-rw-r--r--  1 root      root        3234 Apr  1 08:11 .bashrc
-rw-r--r--  1 root      root           0 Apr  7 10:30 .hidden-file
-rw-r--r--  1 root      root         161 Apr 22  2024 .profile
drwx------  2 root      root        4096 Apr  1 08:10 .ssh
drwxr-xr-x  2 root      root        4096 Apr  1 08:11 .theia
-rw-r--r--  1 root      root         109 Apr  1 08:11 .vimrc
-rw-r--r--  1 root      root         165 Apr  1 08:11 .wget-hsts
-rw-r--r--  1 root      root      523714 Apr  7 10:30 File-01.txt
-rw-r--r--  1 root      root           0 Apr  7 10:30 file-01
-rw-r--r--  1 root      root          18 Apr  7 10:30 file-01.txt
-rw-r--r--  1 root      root           0 Apr  7 10:30 file-02
lrwxrwxrwx  1 root      root           1 Apr  1 08:11 filesystem -> /
-rw-r--r--  1 testuser  testuser      18 Apr  7 10:30 notmyfile
-rw-r--r--  1 otheruser otheruser     19 Apr  7 10:30 notmyfile2
drwxr-xr-x  2 root      root        4096 Apr  7 10:30 testDir
drwxr-xr-x  3 root      root        4096 Apr  7 10:30 testdir
