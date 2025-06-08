# Kubernetes Kind Cluster and Dashboard Setup on macOS

This guide walks you through setting up a local Kubernetes cluster using Kind (Kubernetes IN Docker) on macOS and deploying the Kubernetes Dashboard for UI access.

---

## 🧰 Prerequisites

- macOS system
- Docker Desktop installed and running: https://www.docker.com/products/docker-desktop/
- Homebrew installed: https://brew.sh/

---

## 1. Install Kind

```bash
brew install kind
```

---

## 2. Install kubectl

Follow the official guide:

👉 https://kubernetes.io/docs/tasks/tools/install-kubectl-macos/

Or install via Homebrew:

```bash
brew install kubectl
```

---

## 3. Create a Kind Cluster

```bash
kind create cluster
```

To verify the cluster is up:

```bash
kubectl cluster-info --context kind-kind
```

---

# 📊 Kubernetes Dashboard Setup Guide

This section explains how to install, access, and authenticate the Kubernetes Dashboard.

---

## 4. Install the Kubernetes Dashboard

Apply the official recommended deployment:

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml
```

---

## 5. Start the Dashboard Proxy

Start a local proxy to access the dashboard:

```bash
kubectl proxy
```

Then, open the following URL in your browser:

```
http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/
```

---

## 6. Create an Admin Service Account

Create a file named `admin-user.yaml` with this content:

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard
```

Apply the configuration:

```bash
kubectl apply -f admin-user.yaml
```

---

## 7. Get the Login Token

Run this to generate a login token:

```bash
kubectl -n kubernetes-dashboard create token admin-user
```

Copy the output and use it to log in to the dashboard.

---

## 8. Optional: Secure External Access

For more advanced access options (e.g., Ingress or NodePort), configure your Kind cluster networking carefully and consider TLS + auth middleware.

---

## 🔐 Notes

- Dashboard only works via `https`, even when proxied.
- For production environments, restrict access and audit roles using RBAC.
