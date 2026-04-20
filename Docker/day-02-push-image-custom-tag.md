# Day 02 — Docker: push image with custom tag

**Platform:** KillerCoda  
**Date:** April 2026  
**Topic:** Docker fundamentals — custom versioned image tags

---

## What is image tagging?

By default, Docker uses `:latest` tag when no 
tag is specified. In production environments, 
using `:latest` is considered bad practice because:

- You cannot tell WHICH version is running
- `:latest` can change unexpectedly when a new 
  image is pushed
- Rollbacks become impossible if everything 
  is tagged `:latest`

Versioned tags like `:v1`, `:v1.2.0`, `:2024-04` 
give you full control and traceability.

---

## Commands practised

| Command | What it does |
|---------|-------------|
| `docker tag pinger pinger:v1` | Add `:v1` tag to the local pinger image |
| `docker tag pinger local-registry:5000/pinger:v1` | Tag with registry address AND version |
| `docker image ls` | Confirm all three tags exist on same image |
| `docker push local-registry:5000/pinger:v1` | Push specifically the `:v1` tagged image |

---

## Step by step — what was done

```bash
