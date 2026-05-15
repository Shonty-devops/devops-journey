# Day 02 — Kubernetes: creating a deployment

**Platform:** KillerCoda  
**Date:** April 2026  
**Topic:** Kubernetes fundamentals — what is a deployment and how to create one

---

## What is a Deployment?

A Deployment is a Kubernetes object that describes 
the desired state of a set of Pods. Once created, 
Kubernetes controllers work to ensure the actual 
state matches the desired state — automatically.

Key facts about Deployments:
- Manages and maintains a desired number of Pod replicas
- Automatically replaces failed or deleted Pods
- Enables rolling updates and rollbacks
- Sits above ReplicaSets in the hierarchy

Official docs: https://kubernetes.io/docs/concepts/workloads/controllers/deployment/

---

## Kubernetes object hierarchy

```
Deployment          ← desired state definition
     │
     ▼
 ReplicaSet         ← ensures correct number of pods
     │
     ▼
   Pod              ← actual running container(s)
     │
     ▼
 Container          ← Docker container inside pod
```

---

## Task — create deployment `my-first-deployment`

**Goal:** Create a deployment called 
`my-first-deployment` using `nginx:alpine` 
in the default namespace.

---

## Commands used

```bash
kubectl create deployment my-first-deployment --image=nginx:alpine
# Output: deployment.apps/my-first-deployment created
```

---

## What was tried and what failed

```bash
kubectl inspect my-first-deployment
# Error: unknown command "inspect" for "kubectl"
# kubectl has no "inspect" command
# The correct command is "describe" or "get"
```

### Why it failed

```
Docker uses:     docker inspect
Kubernetes uses: kubectl describe
                 kubectl get

Common mistake when coming from Docker background —
the equivalent of docker inspect in kubectl is
kubectl describe
```

---

## Correct verification commands

```bash
# Quick health check
kubectl get deployment
# Output:
# NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
# my-first-deployment   1/1     1            1           39s
#                       ^^^
#                       1/1 = 1 ready out of 1 desired = healthy

# Detailed information
kubectl describe deployment my-first-deployment
# Shows: replicas, image, strategy, events, conditions
```

---

## Understanding `kubectl get deployment` output

```
NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
my-first-deployment   1/1     1            1           39s

NAME        = deployment name
READY       = ready replicas / desired replicas (1/1 = healthy)
UP-TO-DATE  = replicas updated to latest desired state
AVAILABLE   = replicas available to serve traffic
AGE         = how long deployment has existed
```

---

## Mistake learned today

| Wrong command | Error | Correct command |
|--------------|-------|-----------------|
| `kubectl inspect my-first-deployment` | Unknown command | `kubectl describe deployment my-first-deployment` |

---

## Deployment creation reference

```bash
# Basic deployment
kubectl create deployment <name> --image=<image>

# Deployment with specific replicas
kubectl create deployment <name> \
  --image=<image> \
  --replicas=3

# Deployment with specific port
kubectl create deployment <name> \
  --image=<image> \
  --port=80

# Verify deployment
kubectl get deployment
kubectl get deployment <name>
kubectl describe deployment <name>

# See pods created by deployment
kubectl get pods
```

---

## Key takeaways for DevOps

- Deployments are the standard way to run 
  applications in Kubernetes — not raw pods
- A Deployment creates a ReplicaSet which 
  creates and manages the actual Pods
- `1/1` in the READY column means healthy — 
  desired replicas match running replicas
- `kubectl describe` is the Kubernetes 
  equivalent of `docker inspect`
- Deleting a Pod managed by a Deployment 
  automatically creates a replacement — 
  Deployments are self-healing

---

## New commands learned today

| Command | Purpose |
|---------|---------|
| `kubectl create deployment <name> --image=<img>` | Create a deployment |
| `kubectl get deployment` | List all deployments |
| `kubectl get deployment <name>` | Check specific deployment |
| `kubectl describe deployment <name>` | Full deployment details |

---
- Delete a deployment
- `kubectl apply -f deployment.yaml` — 
  create deployments from YAML files
