# Day 01 — Kubernetes: creating a pod

**Platform:** KillerCoda  
**Date:** April 2026  
**Topic:** Kubernetes fundamentals — what is a pod and how to create one

---

## What is a Pod?

A Pod is the smallest deployable unit in Kubernetes.
Just like a container is the smallest unit in Docker,
a Pod is the smallest unit in Kubernetes.

Key facts about Pods:
- A Pod wraps one or more containers
- All containers in a Pod share the same network
  and storage
- Pods are ephemeral — they are not self-healing
- Kubernetes manages Pods through controllers
  like Deployments and ReplicaSets

Official docs: https://kubernetes.io/docs/concepts/workloads/pods

---

## Relationship — Docker vs Kubernetes units

```
Docker:       Container  ← smallest unit
                 │
Kubernetes:    Pod       ← wraps containers
                 │
              ReplicaSet ← manages multiple pods
                 │
              Deployment ← manages ReplicaSets
```

---

## Task — create a pod named `my-pod`

**Goal:** Create a pod called `my-pod` using
the `nginx:alpine` image.

---

## What was tried and what failed

```bash
kubectl create pod my-pod nginx:alpine
# Error: Unexpected args: [pod my-pod nginx:alpine]
# See 'kubectl create -h' for help and examples
```

### Why it failed

```
kubectl create pod = NOT a valid subcommand for pods
                     kubectl create is used for resources
                     from YAML files or specific types
                     Use kubectl run for quick pod creation
```

---

## Correct command

```bash
kubectl run my-pod --image=nginx:alpine
# Output: pod/my-pod created

# Breaking it down:
# kubectl run          = create and run a pod directly
# my-pod               = name of the pod
# --image=nginx:alpine = container image to use
```

---

## Verify pod was created

```bash
kubectl get pods
# Output:
# NAME      READY   STATUS    RESTARTS   AGE
# my-pod    1/1     Running   0          10s

kubectl describe pod my-pod
# Shows full details: image, node, IP, events
```

---

## kubectl command structure

```
kubectl  <verb>   <resource-type>  <name>      <flags>
kubectl  run      (pod created)    my-pod      --image=nginx:alpine
kubectl  get      pod              my-pod
kubectl  describe pod              my-pod
```

---

## Mistake learned today

| Wrong command | Error | Correct command |
|--------------|-------|-----------------|
| `kubectl create pod my-pod nginx:alpine` | Unexpected args | `kubectl run my-pod --image=nginx:alpine` |

---

## Quick pod creation reference

```bash
# Basic pod
kubectl run <name> --image=<image>

# Pod with specific port
kubectl run <name> --image=<image> --port=80

# List all pods
kubectl get pods

# List pods with more details
kubectl get pods -o wide

# Describe pod — full details and events
kubectl describe pod <name>

# Get pod logs
kubectl logs <name>

# Get shell inside pod
kubectl exec -it <name> -- sh
```

---

## Key takeaways for DevOps

- `kubectl run` is the quick way to create a
  single pod — similar to `docker run`
- `kubectl create` is used for creating resources
  from YAML files — NOT for raw pod creation
- In production, pods are rarely created directly —
  Deployments manage pods automatically with
  self-healing and scaling
- The default namespace is used when no namespace
  flag is specified

---

## New commands learned today

| Command | Purpose |
|---------|---------|
| `kubectl run <name> --image=<img>` | Create a pod quickly |
| `kubectl get pods` | List all pods in current namespace |
| `kubectl get pods -o wide` | List pods with extra details |
| `kubectl describe pod <name>` | Full pod details and event log |
| `kubectl logs <name>` | View pod container logs |

---
  definition as YAML
- Deployments — manage multiple pod replicas
  with self-healing
