# Day 10 — Docker: reattaching volume to a new container

**Platform:** KillerCoda  
**Date:** April 2026  
**Topic:** Docker storage — proving volume data persists across containers

---

## What does this lesson demonstrate?

This is the final proof of named volume persistence. 
The old `sample-app` container was removed in Day 22. 
Now we spin up a BRAND NEW container attaching 
the SAME `sample-volume` — and the updated content 
from Day 22 is immediately available.

No rebuild. No data restore. Just attach and serve.

---

## Commands practised

```bash
