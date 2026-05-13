# Day 10 — Docker: updating data in a named volume

**Platform:** KillerCoda  
**Date:** April 2026  
**Topic:** Docker storage — modifying volume content and data persistence

---

## What does this lesson demonstrate?

Unlike bind mounts where you edit files on the 
HOST, named volume data lives in Docker's 
managed storage. To update content you either:

1. Write via `docker exec` inside the container
2. Write directly to the volume mountpoint on host

This lesson uses `docker exec` — the standard 
approach for named volumes.

---

## Commands practised

```bash
