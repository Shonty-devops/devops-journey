# Day 09 — Docker: overriding environment variables at runtime

**Platform:** KillerCoda  
**Date:** April 2026  
**Topic:** Docker fundamentals — injecting and overriding ENV at runtime

---

## What does this lesson demonstrate?

Using the SAME image (`sample-image`) from Day 19 
without touching the Dockerfile, we:

- Override an existing variable (`key1=new-value1`)
- Add a brand new variable (`key2=value2`)

This proves the power of runtime configuration — 
one image, infinite configurations.

---

## Commands practised

```bash
