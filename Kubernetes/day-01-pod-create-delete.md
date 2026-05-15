# Day 01 — Kubernetes: creating and deleting pods

**Platform:** KillerCoda  
**Date:** April 2026  
**Topic:** Kubernetes fundamentals — pods introduction, create and delete

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

## Task 1 — Create a pod

**Goal:** Create a pod called `my-pod` using 
the `nginx:alpine` image.

---

### What was tried and what failed

```bash
# WRONG — incorrect command syntax
kubectl create pod my-pod nginx:alpine
# Error: Unexpected args: [pod my-pod nginx:alpine]
# See 'kubectl create -h' for help and examples

# kubectl create pod does not work this way
# "create" is used for resources from YAML files
# or specific resource types with their own flags
```

### Why it failed

```
kubectl create    = create resources from file or specific syntax
kubectl create pod = NOT a valid subcommand for pods this way
                     use kubectl run instead for quick pod creation
```

---

### Correct command

```bash
kubectl run my-pod --image=nginx:alpine
# Output: pod/my-pod created

# Breaking it down:
# kubectl run        = create and run a pod directly
# my-pod             = name of the pod
# --image=nginx:alpine = container image to use
```

---

### Verify pod was created

```bash
kubectl get pods
# Output:
# NAME      READY   STATUS    RESTARTS   AGE
# my-pod    1/1     Running   0          10s

kubectl describe pod my-pod
# Shows full details: image, node, IP, events
```

---

## Task 2 — Delete a pod

**Goal:** Delete the pod called `my-pod`.

---

### What was tried and what failed

```bash
# WRONG — missing resource type
kubectl delete my-pod
# Error: the server doesn't have a resource type "my-pod"

# kubectl needs to know WHAT TYPE of resource
# to delete — just the name is not enough
```

### Why it failed

```
kubectl delete <resource-type> <name>

# Kubernetes has many resource types:
# pods, deployments, services, configmaps, etc.
# You must specify which type you are deleting
# "my-pod" alone is just a name — could be anything
```

---

### Correct command

```bash
kubectl delete pod my-pod
# Output: pod "my-pod" deleted from default namespace

# Breaking it down:
# kubectl delete  = delete a resource
# pod             = resource type to delete
# my-pod          = name of the specific pod
```

---

### Verify pod was deleted

```bash
kubectl get pods
# Output:
# No resources found in default namespace.
# or pod shows "Terminating" briefly then disappears
```

---

## Common mistakes learned today

| Wrong command | Error | Correct command |
|--------------|-------|-----------------|
| `kubectl create pod my-pod nginx:alpine` | Unexpected args | `kubectl run my-pod --image=nginx:alpine` |
| `kubectl delete my-pod` | No resource type found | `kubectl delete pod my-pod` |

---

## kubectl command structure

```
kubectl  <verb>   <resource-type>  <name>      <flags>
kubectl  run      (pod created)    my-pod      --image=nginx:alpine
kubectl  get      pod              my-pod
kubectl  delete   pod              my-pod
kubectl  describe pod              my-pod

Verbs:    run, get, create, delete, describe, apply, edit
Types:    pod, deployment, service, configmap, secret, node
```

---

## Quick pod commands reference

```bash
# Create pod
kubectl run <name> --image=<image>

# Create pod with specific port
kubectl run <name> --image=<image> --port=80

# Create pod and expose port
kubectl run <name> --image=<image> --port=80 \
  --expose

# List all pods
kubectl get pods

# List pods with more details
kubectl get pods -o wide

# Describe pod — full details and events
kubectl describe pod <name>

# Delete pod
kubectl delete pod <name>

# Delete pod immediately without grace period
kubectl delete pod <name> --grace-period=0 --force

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
  from YAML files or specific resource types — 
  NOT for raw pod creation this way
- Always specify the resource TYPE before the 
  name in `kubectl delete` — this applies to 
  every Kubernetes resource, not just pods
- In production, pods are rarely created directly — 
  Deployments manage pods automatically with 
  self-healing and scaling capabilities
- The default namespace is used when no namespace 
  flag is specified — equivalent to working in 
  the root directory

---

## Namespaces — brief intro

```bash
# Default namespace used unless specified
kubectl get pods                        # default namespace
kubectl get pods -n kube-system         # kube-system namespace
kubectl get pods --all-namespaces       # all namespaces

# Delete pod in specific namespace
kubectl delete pod my-pod -n my-namespace
```

---

## New commands learned today

| Command | Purpose |
|---------|---------|
| `kubectl run <name> --image=<img>` | Create a pod quickly |
| `kubectl get pods` | List all pods in current namespace |
| `kubectl get pods -o wide` | List pods with extra details |
| `kubectl describe pod <name>` | Full pod details and event log |
| `kubectl delete pod <name>` | Delete a specific pod |
| `kubectl logs <name>` | View pod container logs |

---
- ReplicaSets — ensure a specified number 
  of pod replicas are running
