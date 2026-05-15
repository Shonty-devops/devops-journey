# Day 02 — Kubernetes: deleting a pod

**Platform:** KillerCoda  
**Date:** April 2026  
**Topic:** Kubernetes fundamentals — how to correctly delete a pod

---

## Task — delete the pod named `my-pod`

**Goal:** Delete the pod called `my-pod` from
the default namespace.

---

## What was tried and what failed

```bash
kubectl delete my-pod
# Error: the server doesn't have a resource type "my-pod"
```

### Why it failed

```
kubectl delete requires TWO things:
1. Resource TYPE  (pod, deployment, service...)
2. Resource NAME  (my-pod)

"my-pod" alone is just a name — Kubernetes
does not know WHAT TYPE of resource to delete.
You must always specify the resource type first.
```

---

## Correct command

```bash
kubectl delete pod my-pod
# Output: pod "my-pod" deleted from default namespace

# Breaking it down:
# kubectl delete = delete a resource
# pod            = resource TYPE to delete
# my-pod         = name of the specific pod
```

---

## Verify pod was deleted

```bash
kubectl get pods
# Output:
# No resources found in default namespace.
# Pod may briefly show "Terminating" before disappearing
```

---

## kubectl delete command structure

```
kubectl  delete  <resource-type>  <name>
kubectl  delete  pod              my-pod
kubectl  delete  deployment       my-deployment
kubectl  delete  service          my-service
kubectl  delete  configmap        my-config
```

Resource type is ALWAYS required before the name.

---

## Mistake learned today

| Wrong command | Error | Correct command |
|--------------|-------|-----------------|
| `kubectl delete my-pod` | No resource type found | `kubectl delete pod my-pod` |

---

## Pod deletion reference

```bash
# Delete specific pod
kubectl delete pod <name>

# Delete pod immediately — no grace period
kubectl delete pod <name> --grace-period=0 --force

# Delete pod in specific namespace
kubectl delete pod <name> -n <namespace>

# Delete all pods in current namespace
kubectl delete pods --all

# Delete pod using its YAML file
kubectl delete -f pod.yaml
```

---

## What happens when a pod is deleted

```
kubectl delete pod my-pod
        │
        ▼
Kubernetes sends SIGTERM to container
        │
        ▼
Grace period starts (default 30 seconds)
        │
        ▼
Container shuts down gracefully
        │
        ▼
Pod removed from cluster
        │
        ▼
kubectl get pods → No resources found
```

If container does not stop within grace period,
Kubernetes sends SIGKILL to force terminate it.

---

## Namespaces and deletion

```bash
# Default namespace (no flag needed)
kubectl delete pod my-pod

# Specific namespace
kubectl delete pod my-pod -n kube-system

# All namespaces — not possible with delete
# Must specify namespace explicitly
kubectl get pods --all-namespaces   # find which namespace
kubectl delete pod my-pod -n <found-namespace>
```

---

## Key takeaways for DevOps

- Always specify resource TYPE before name in
  `kubectl delete` — this applies to every
  Kubernetes resource, not just pods
- Pods deleted directly are gone forever —
  no recycle bin, no undo
- Pods managed by a Deployment are automatically
  recreated after deletion — the Deployment
  controller maintains the desired replica count
- Use `--grace-period=0 --force` only in
  emergencies — always prefer graceful shutdown
  in production to avoid data corruption

---

## New commands learned today

| Command | Purpose |
|---------|---------|
| `kubectl delete pod <name>` | Delete a specific pod |
| `kubectl delete pod <name> --force` | Force delete immediately |
| `kubectl delete pods --all` | Delete all pods in namespace |
| `kubectl delete -f pod.yaml` | Delete resource defined in YAML |

---
