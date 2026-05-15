# Day 02 — Kubernetes: updating deployment image

**Platform:** KillerCoda  
**Date:** April 2026  
**Topic:** Kubernetes fundamentals — rolling update with new image

---

## What is a rolling update?

A rolling update gradually replaces old pods 
with new ones running the updated image. 
Traffic is never interrupted — some pods 
always remain available during the update.

```
Rolling update: nginx:alpine → httpd:alpine

Step 1:  [nginx] [nginx]        ← 2 running
Step 2:  [httpd] [nginx]        ← 1 updated, 1 old
Step 3:  [httpd] [httpd]        ← update complete
```

Default strategy: `rollingUpdate`
Official docs: https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#scaling-a-deployment

---

## Task — update image from `nginx:alpine` to `httpd:alpine`

**Goal:** Change the running image of 
`my-first-deployment` without downtime.

---

## What was tried and what failed

```bash
kubectl set image deployment/my-first-deployment nginx:alpine=httpd:alpine
# Error: unable to find container named "nginx:alpine"
```

### Why it failed

```
kubectl set image format:
deployment/<name> <container-name>=<new-image>

The container name is NOT the image name.
Container name is set when deployment is created.
When using kubectl create deployment, the container
name defaults to the IMAGE NAME without the tag.

nginx:alpine → container name is "nginx" (not "nginx:alpine")
```

---

## Correct command

```bash
kubectl set image deployment/my-first-deployment nginx=httpd:alpine
# Output: deployment.apps/my-first-deployment image updated

# Breaking it down:
# deployment/my-first-deployment = resource type/name
# nginx                          = container name (not image name)
# httpd:alpine                   = new image to use
```

---

## Verify update was applied

```bash
kubectl get deployment
# Output:
# NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
# my-first-deployment   2/2     2            2           8m17s

# Check which image is now running
kubectl describe deployment my-first-deployment | grep Image
# Image: httpd:alpine  ← confirmed updated
```

---

## Finding the correct container name

```bash
# Before updating, always check container name first
kubectl describe deployment my-first-deployment | grep -A 2 "Containers"
# Output:
#   Containers:
#    nginx:           ← this is the container name
#     Image: nginx:alpine

# Container name = "nginx"
# NOT "nginx:alpine"
```

---

## Mistake learned today

| Wrong command | Error | Correct command |
|--------------|-------|-----------------|
| `kubectl set image ... nginx:alpine=httpd:alpine` | Container not found | `kubectl set image ... nginx=httpd:alpine` |

---

## Image update reference

```bash
# Update image
kubectl set image deployment/<name> \
  <container-name>=<new-image>

# Watch rolling update progress
kubectl rollout status deployment/<name>

# Check update history
kubectl rollout history deployment/<name>

# Rollback to previous image
kubectl rollout undo deployment/<name>

# Verify new image
kubectl describe deployment <name> | grep Image
```

---

## Rolling update process

```
kubectl set image deployment/my-first-deployment nginx=httpd:alpine
        │
        ▼
Kubernetes creates new ReplicaSet with httpd:alpine
        │
        ▼
New pods start with httpd:alpine
        │
        ▼
Old pods with nginx:alpine are terminated
        │
        ▼
All pods now running httpd:alpine
        │
        ▼
Old ReplicaSet kept (for rollback capability)
```

---

## Key takeaways for DevOps

- Container name ≠ image name — always check 
  with `kubectl describe` before `kubectl set image`
- Rolling updates cause zero downtime — 
  always some pods available during update
- Old ReplicaSet is kept after update — 
  this enables instant rollback with 
  `kubectl rollout undo`
- In production, always watch rollout progress:
  `kubectl rollout status deployment/<name>`

---

## New commands learned today

| Command | Purpose |
|---------|---------|
| `kubectl set image deployment/<n> <c>=<img>` | Update container image |
| `kubectl rollout status deployment/<name>` | Watch update progress |
| `kubectl rollout undo deployment/<name>` | Rollback to previous image |
| `kubectl describe deployment <n> \| grep Image` | Verify current image |

---
- `kubectl rollout undo` — rollback updates
- Delete deployment cleanly
- YAML-based deployments with `kubectl apply`
