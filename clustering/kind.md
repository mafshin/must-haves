# Kubernetes Kind Cluster and Dashboard Setup (macOS & Windows WSL2)

This guide walks you through setting up a local Kubernetes cluster using Kind (Kubernetes IN Docker) and deploying the Kubernetes Dashboard for UI access on both macOS and Windows WSL2 environments.

---

## 🧰 Prerequisites

### For macOS
- Docker Desktop installed and running: https://www.docker.com/products/docker-desktop/
- Homebrew installed: https://brew.sh/

### For Windows (WSL2)
- Windows 10/11 with WSL2 enabled
- Ubuntu (or Debian-based) WSL2 distro installed
- Docker Desktop for Windows (with WSL2 integration): https://www.docker.com/products/docker-desktop/
- Docker integration enabled for your WSL2 distro

---

## 1. Install Kind

### On macOS

```bash
brew install kind
```

### On Windows (WSL2)

```bash
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.22.0/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind
```

---

## 2. Install kubectl

### On macOS

```bash
brew install kubectl
```

Or follow the official guide:  
👉 https://kubernetes.io/docs/tasks/tools/install-kubectl-macos/

### On Windows (WSL2)

```bash
sudo apt update && sudo apt install -y apt-transport-https ca-certificates curl
sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt update
sudo apt install -y kubectl
```

---

## 3. Create a Kind Cluster

Make sure Docker is running.

```bash
kind create cluster
```

Verify the cluster is running:

```bash
kubectl cluster-info --context kind-kind
```

---

# 📊 Kubernetes Dashboard Setup Guide

---

## 4. Install the Kubernetes Dashboard

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml
```

---

## 5. Start the Dashboard Proxy

```bash
kubectl proxy
```

Then open this in your browser:

```
http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/
```

> **Note:** On Windows WSL2, open this URL in your **Windows browser**, not the WSL2 browser.

---

## 6. Create an Admin Service Account

Create a file named `admin-user.yaml` with the following content:

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

Generate a token to log in:

```bash
kubectl -n kubernetes-dashboard create token admin-user
```

Paste this token in the Dashboard login screen.

---

## 🔐 Notes

- The Dashboard runs over HTTPS internally.
- Use RBAC policies to secure access.
- For Windows WSL2, access the dashboard from the Windows browser.
- For production access, consider Ingress with TLS and authentication middleware.

# ✅  Using the Cluster

## Running a Pod
To run a simple Alpine-based pod in your Kubernetes cluster (like in Kind), follow these steps:

```
kubectl run alpine --image=alpine --restart=Never --command -- sleep 3600
```
This creates a temporary pod named alpine using the alpine image that sleeps for 1 hour.

Then, to open a shell inside it:
```
kubectl exec -it alpine -- sh
```
If sh doesn't work, try ash (Alpine shell).
