# Day 03 — Kubernetes: dashboard setup and access

**Platform:** KillerCoda  
**Date:** April 2026  
**Topic:** Kubernetes fundamentals — installing and accessing the K8s dashboard

---

## What is the Kubernetes Dashboard?

The Kubernetes Dashboard is a web-based UI for 
managing and monitoring Kubernetes clusters. 
It allows you to:
- View all cluster resources visually
- Deploy and manage applications
- Monitor pod logs and resource usage
- Manage storage, networking, and configs

---

## Key requirements for this setup

```
✓ Service must bind to 0.0.0.0 (all interfaces)
  NOT just localhost — needed for external access

✓ Service must use HTTP not HTTPS
  Insecure login enabled for lab environment

✓ Port-forward must also bind to 0.0.0.0
  So the dashboard is reachable from outside
```

---

## Step 1 — install the dashboard

```bash
# Apply customised dashboard YAML
kubectl apply -f /root/dashboard.yaml

# Wait for all dashboard pods to be ready
kubectl -n kubernetes-dashboard wait \
  --for=condition=ready pod --all
# Output: pod/kubernetes-dashboard-xxx condition met
```

---

## Key modifications in dashboard.yaml

### Args added to dashboard container

```yaml
args:
  - --namespace=kubernetes-dashboard
  - --enable-skip-login
  - --disable-settings-authorizer
  - --enable-insecure-login
  - --insecure-bind-address=0.0.0.0
```

| Argument | Purpose |
|----------|---------|
| `--namespace=kubernetes-dashboard` | Restrict dashboard to its namespace |
| `--enable-skip-login` | Allow access without token (lab only) |
| `--disable-settings-authorizer` | Skip settings auth check |
| `--enable-insecure-login` | Allow HTTP login |
| `--insecure-bind-address=0.0.0.0` | Bind to all interfaces not just localhost |

### Updated Service definition

```yaml
kind: Service
apiVersion: v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kubernetes-dashboard
spec:
  ports:
    - port: 9090
      targetPort: 9090
  selector:
    k8s-app: kubernetes-dashboard
```

Key points:
- Port `9090` used instead of default `443`
- HTTP instead of HTTPS
- Selector targets `k8s-app: kubernetes-dashboard` pods

---

## Step 2 — create ServiceAccount and token

```bash
# Create admin service account in dashboard namespace
kubectl -n kubernetes-dashboard create sa admin-user
# Output: serviceaccount/admin-user created

# Bind admin-user to cluster-admin role
kubectl create clusterrolebinding admin-user \
  --clusterrole cluster-admin \
  --serviceaccount kubernetes-dashboard:admin-user
# Output: clusterrolebinding.rbac.authorization.k8s.io/admin-user created

# Generate access token
kubectl -n kubernetes-dashboard create token admin-user
# Output: eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...
# Copy this token — needed to log into dashboard
```

---

## Understanding the RBAC setup

```
ServiceAccount (admin-user)
        │
        │ bound via ClusterRoleBinding
        │
        ▼
ClusterRole (cluster-admin)
        │
        └── Full access to ALL resources
            in ALL namespaces

This gives the dashboard token admin-level
access to view and manage everything in the cluster.
```

---

## Step 3 — port-forward the dashboard

```bash
kubectl -n kubernetes-dashboard port-forward \
  service/kubernetes-dashboard \
  9090:9090 \
  --address 0.0.0.0
# --address 0.0.0.0 = bind to all interfaces
# Without this, port-forward only works on localhost
# 9090:9090 = local port : service port
```

---

## Step 4 — access the dashboard

After running port-forward:

1. Open the dashboard URL in your browser
2. Select "Token" login method
3. Paste the token generated in Step 2
4. Click Sign In

```
Dashboard available at:
http://<host-ip>:9090

Or via KillerCoda traffic port link
```

---

## What you can do in the dashboard

```
Workloads tab:
├── Deployments   — view, scale, update
├── Pods          — logs, exec, delete
├── ReplicaSets   — view replica status
└── StatefulSets  — manage stateful apps

Services tab:
├── Services      — view endpoints
├── Ingresses     — view routing rules
└── Endpoints     — view pod IPs

Config tab:
├── ConfigMaps    — view configuration
└── Secrets       — view secret names

Storage tab:
└── PersistentVolumeClaims — view storage
```

---

## Key concepts learned today

### Namespace: kubernetes-dashboard

```bash
# All dashboard resources live in their own namespace
kubectl get all -n kubernetes-dashboard
# Shows: pods, services, deployments in dashboard namespace

# Check dashboard pods are running
kubectl get pods -n kubernetes-dashboard
```

### ServiceAccount

```bash
# ServiceAccount = identity for processes in pods
# Used by dashboard to authenticate API requests
kubectl get sa -n kubernetes-dashboard
# Shows: default, admin-user
```

### ClusterRoleBinding

```bash
# Grants admin-user full cluster access
kubectl get clusterrolebinding admin-user
# Shows the binding between SA and cluster-admin role
```

---

## Port-forward vs Service types

```
kubectl port-forward (what we used):
├── Temporary — stops when terminal closes
├── Good for: development, debugging, lab access
└── Binds to localhost by default (use --address 0.0.0.0 for external)

NodePort Service:
├── Permanent — always accessible
├── Good for: lab and development clusters
└── Exposes service on every node's IP

LoadBalancer Service:
├── Permanent — cloud load balancer created
├── Good for: production on cloud providers
└── Gets external IP automatically
```

---

## Key takeaways for DevOps

- Kubernetes Dashboard is a powerful visual 
  tool for cluster management — faster than 
  typing `kubectl` for routine checks
- In production, dashboard access should be 
  strictly controlled — use proper TLS and 
  restrict ServiceAccount permissions
- `--insecure-bind-address=0.0.0.0` is for 
  lab use ONLY — never use in production
- `cluster-admin` ClusterRole gives full 
  access — in production create restricted 
  roles with only necessary permissions
- `kubectl port-forward` is the standard way 
  to access internal services during development 
  and debugging — you will use it constantly

---

## Commands reference

```bash
# Apply dashboard YAML
kubectl apply -f /root/dashboard.yaml

# Wait for pods to be ready
kubectl -n kubernetes-dashboard wait \
  --for=condition=ready pod --all

# Create service account
kubectl -n kubernetes-dashboard create sa admin-user

# Create cluster role binding
kubectl create clusterrolebinding admin-user \
  --clusterrole cluster-admin \
  --serviceaccount kubernetes-dashboard:admin-user

# Generate token
kubectl -n kubernetes-dashboard create token admin-user

# Port forward dashboard
kubectl -n kubernetes-dashboard port-forward \
  service/kubernetes-dashboard 9090:9090 --address 0.0.0.0

# Check dashboard resources
kubectl get all -n kubernetes-dashboard
kubectl get pods -n kubernetes-dashboard
kubectl get sa -n kubernetes-dashboard
```

---

## New commands learned today

| Command | Purpose |
|---------|---------|
| `kubectl apply -f <file>` | Apply resources from YAML file |
| `kubectl -n <ns> wait --for=condition=ready pod --all` | Wait for all pods to be ready |
| `kubectl -n <ns> create sa <name>` | Create a ServiceAccount |
| `kubectl create clusterrolebinding` | Bind role to ServiceAccount |
| `kubectl -n <ns> create token <sa>` | Generate auth token for ServiceAccount |
| `kubectl port-forward service/<n> <port> --address 0.0.0.0` | Forward service port to host |
| `kubectl get all -n <namespace>` | List all resources in a namespace |

---
