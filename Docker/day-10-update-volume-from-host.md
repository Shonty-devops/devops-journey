# Day 10 — Docker: updating volume data directly from host

**Platform:** KillerCoda  
**Date:** April 2026  
**Topic:** Docker storage — editing named volume data on the host filesystem

---

## What does this lesson demonstrate?

Named volumes are stored on the host at a 
specific path managed by Docker. You can find 
this path and edit files DIRECTLY on the host — 
without `docker exec` or going inside the container.

This completes the full picture of how named 
volumes connect host and container storage.

---

## Commands practised

```bash
