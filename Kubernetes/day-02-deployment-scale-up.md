# Day 02 — Kubernetes: scaling a deployment up

**Platform:** KillerCoda  
**Date:** April 2026  
**Topic:** Kubernetes fundamentals — scaling deployments to more replicas

---

## What is scaling in Kubernetes?

Scaling means changing the number of Pod replicas 
running for a Deployment. Scaling UP increases 
replicas to handle more traffic or load.

```
Before scaling:          After scaling up:
┌─────────────┐          ┌─────────────┐
│  Deployment │          │  Deployment │
│  replicas:1 │          │  replicas:3 │
│             │          │             │
│  ┌───────┐  │          │  ┌───────┐  │
│  │ Pod 1 │  │          │  │ Pod 1 │  │
│  └───────┘  │          │  ├───────┤  │
└─────────────┘          │  │ Pod 2 │  │
                         │  ├───────┤  │
                         │  │ Pod 3 │  │
                         │  └───────┘  │
                         └─────────────┘
```

Official docs: https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#scaling-a-deployment

---

## Task — scale `my-first-deployment` to 3 replicas

**Goal:** Scale the existing deployment up 
from 1 replica to 3 replicas.

---

## Commands used

```bash
# Scale up to 3 replicas
kubectl scale deployment my-first-deployment --replicas=3
# Output: deployment.apps/my-first-deployment scaled

# Verify all 3 replicas are ready
kubectl get deployment
# Output:
# NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
# my-first-deployment   3/3     3            3           3m2s
#                       ^^^
#                       3/3 = all 3 replicas ready = healthy
```

---

## Verifying pods were created

```bash
kubectl get pods
# Output:
# NAME                                   READY   STATUS    RESTARTS
# my-first-deployment-abc123-def45       1/1     Running   0
# my-first-deployment-abc123-ghi67       1/1     Running   0
# my-first-deployment-abc123-jkl89       1/1     Running   0
# 3 pods — one per replica
```

---

## Understanding scaling output

```
READY   UP-TO-DATE   AVAILABLE
3/3     3            3

READY       = 3 out of 3 desired replicas are running
UP-TO-DATE  = all 3 updated to latest spec
AVAILABLE   = all 3 ready to serve traffic
```

---

## Why scale up?

```
High traffic        → more replicas = more capacity
Node failure        → other replicas keep app running
Zero downtime deploy → scale up before updating
Load balancing      → traffic spread across all replicas
```

---

## Scaling reference

```bash
# Scale up
kubectl scale deployment <name> --replicas=<number>

# Check replica count
kubectl get deployment <name>

# Watch scaling in real time
kubectl get pods -w

# Check deployment details
kubectl describe deployment <name>
```

---

## Key takeaways for DevOps

- Kubernetes scaling is instant — new pods 
  start immediately after scale command
- All replicas share traffic automatically 
  via Kubernetes Services
- Scaling does not cause downtime — existing 
  pods keep running while new ones start
- In production, use Horizontal Pod Autoscaler 
  (HPA) for automatic scaling based on CPU/memory

---

## New commands learned today

| Command | Purpose |
|---------|---------|
| `kubectl scale deployment <name> --replicas=<n>` | Scale deployment to N replicas |
| `kubectl get pods` | Verify pods created by scaling |
| `kubectl get pods -w` | Watch pods in real time |

---
- Update deployment image
- Horizontal Pod Autoscaler (HPA)
- `kubectl rollout status` — watch rollout progress
