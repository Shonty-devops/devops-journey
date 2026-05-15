# Day 02 — Kubernetes: scaling a deployment down

**Platform:** KillerCoda  
**Date:** April 2026  
**Topic:** Kubernetes fundamentals — reducing deployment replicas

---

## What is scaling down?

Scaling down reduces the number of running Pod 
replicas. Kubernetes gracefully terminates 
excess pods to reach the desired count.

```
Before scaling down:     After scaling down:
┌─────────────┐          ┌─────────────┐
│  replicas:3 │          │  replicas:2 │
│  ┌───────┐  │          │  ┌───────┐  │
│  │ Pod 1 │  │          │  │ Pod 1 │  │
│  ├───────┤  │          │  ├───────┤  │
│  │ Pod 2 │  │          │  │ Pod 2 │  │
│  ├───────┤  │          │  └───────┘  │
│  │ Pod 3 │  │ ───────► │             │
│  └───────┘  │ scale    │  Pod 3      │
└─────────────┘ down     │  terminated │
                         └─────────────┘
```

Official docs: https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#scaling-a-deployment

---

## Task — scale `my-first-deployment` to 2 replicas

**Goal:** Scale the deployment down from 
3 replicas to 2 replicas.

---

## Commands used

```bash
# Scale down to 2 replicas
kubectl scale deployment my-first-deployment --replicas=2
# Output: deployment.apps/my-first-deployment scaled

# Verify 2 replicas are ready
kubectl get deployment
# Output:
# NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
# my-first-deployment   2/2     2            2           3m59s
#                       ^^^
#                       2/2 = both replicas healthy
```

---

## Verify one pod was terminated

```bash
kubectl get pods
# Output:
# NAME                                   READY   STATUS
# my-first-deployment-abc123-def45       1/1     Running
# my-first-deployment-abc123-ghi67       1/1     Running
# Only 2 pods now — third was terminated gracefully
```

---

## Scale up vs scale down — same command

```bash
# Same command — just change the number
kubectl scale deployment my-first-deployment --replicas=3  # up
kubectl scale deployment my-first-deployment --replicas=2  # down
kubectl scale deployment my-first-deployment --replicas=1  # down more
kubectl scale deployment my-first-deployment --replicas=0  # stop all pods
```

---

## Scaling to zero

```bash
# Scale to 0 — stops all pods but keeps deployment
kubectl scale deployment my-first-deployment --replicas=0

kubectl get deployment
# NAME                  READY   UP-TO-DATE   AVAILABLE
# my-first-deployment   0/0     0            0

kubectl get pods
# No resources found — all pods terminated
# Deployment still exists — scale back up anytime
```

---

## Why scale down?

```
Off-peak hours      → fewer replicas = less cost
Resource limits     → free up cluster resources
Maintenance mode    → scale to 0 temporarily
Cost optimisation   → right-size for current load
```

---

## Key takeaways for DevOps

- Scale down is graceful — Kubernetes sends 
  SIGTERM and waits for pods to finish 
  before terminating
- Scaling to 0 effectively pauses an application 
  without deleting the Deployment configuration
- In production, combine manual scaling with 
  Horizontal Pod Autoscaler for dynamic scaling
- Always verify READY column after scaling — 
  `2/2` confirms all desired replicas are healthy

---

## New commands learned today

| Command | Purpose |
|---------|---------|
| `kubectl scale deployment <name> --replicas=<n>` | Scale down to N replicas |
| `kubectl scale deployment <name> --replicas=0` | Stop all pods, keep deployment |
| `kubectl get pods` | Verify correct number of pods running |

---
- Delete deployment entirely
- Horizontal Pod Autoscaler — automatic scaling
- `kubectl rollout pause` — pause a deployment
