# Day 02 — Kubernetes: deleting a deployment

**Platform:** KillerCoda  
**Date:** April 2026  
**Topic:** Kubernetes fundamentals — clean up deployments and verify removal

---

## Why delete a deployment?

Deleting unused deployments frees up cluster 
resources — CPU, memory, and storage. It is 
good practice to clean up resources that are 
no longer needed.

---

## Task — delete `my-first-deployment`

**Goal:** Delete the deployment and confirm 
it no longer exists.

---

## Commands used

```bash
# Delete the deployment
kubectl delete deployment my-first-deployment
# Output: deployment.apps "my-first-deployment" deleted from default namespace

# Verify deployment is gone
kubectl get deployment my-first-deployment
# Error from server (NotFound):
# deployments.apps "my-first-deployment" not found

# Confirm no deployments remain
kubectl get deployment
# No resources found in default namespace.
```

---

## What gets deleted with the deployment

```
kubectl delete deployment my-first-deployment
        │
        ▼
Deployment deleted
        │
        ▼
ReplicaSets deleted (all versions)
        │
        ▼
All Pods terminated and deleted
        │
        ▼
Cluster resources freed
```

Deleting a Deployment cascades down —
ReplicaSets and Pods are all removed automatically.

---

## Deployment vs Pod deletion — key difference

```bash
# Delete a Pod managed by Deployment
kubectl delete pod my-first-deployment-abc123
# Pod is deleted BUT Deployment recreates it immediately
# Deployment self-healing kicks in

# Delete the Deployment itself
kubectl delete deployment my-first-deployment
# Everything is gone — no recreation
# Deployment, ReplicaSet, and all Pods removed
```

---

## Verify complete cleanup

```bash
# Check deployment gone
kubectl get deployment
# No resources found in default namespace.

# Check pods also gone
kubectl get pods
# No resources found in default namespace.

# Check replicasets also gone
kubectl get replicaset
# No resources found in default namespace.
```

---

## Deletion reference

```bash
# Delete specific deployment
kubectl delete deployment <name>

# Delete using YAML file
kubectl delete -f deployment.yaml

# Delete all deployments in namespace
kubectl delete deployments --all

# Delete deployment in specific namespace
kubectl delete deployment <name> -n <namespace>

# Force delete immediately
kubectl delete deployment <name> --grace-period=0
```

---

## Complete deployment lifecycle — summary

```
Day 03: kubectl create deployment my-first-deployment --image=nginx:alpine
        → 1 replica running

Day 04: kubectl scale deployment my-first-deployment --replicas=3
        → 3 replicas running

Day 05: kubectl scale deployment my-first-deployment --replicas=2
        → 2 replicas running

Day 06: kubectl set image deployment/my-first-deployment nginx=httpd:alpine
        → 2 replicas running httpd:alpine

Day 07: kubectl delete deployment my-first-deployment
        → nothing running, cluster clean
```

---

## Key takeaways for DevOps

- Always clean up unused deployments in 
  shared or production clusters — 
  idle pods consume real resources and cost money
- Deleting a Deployment removes everything 
  it manages — no orphaned pods left behind
- `NotFound` error after deletion confirms 
  the resource is completely gone
- In production, use namespaces to organise 
  resources — easier to clean up entire 
  namespaces than individual deployments

---

## New commands learned today

| Command | Purpose |
|---------|---------|
| `kubectl delete deployment <name>` | Delete deployment and all its pods |
| `kubectl delete deployments --all` | Delete all deployments in namespace |
| `kubectl delete -f <file>.yaml` | Delete resources defined in YAML file |
| `kubectl get deployment <name>` | Verify deployment no longer exists |

---
  to internal or external traffic
- Namespaces — organise and isolate resources
- `kubectl rollout` — manage deployment updates
