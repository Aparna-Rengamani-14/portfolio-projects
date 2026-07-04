# Setup Guide: OpenNMS on Kubernetes (Minikube)

This guide walks through setting up the full environment from a Windows machine: WSL2 → Docker → Minikube → kubectl → Postgres → OpenNMS.

## 0. Prerequisites

- Windows with **WSL2** enabled (verify with `wsl -l -v`; version must show `2`)
- All commands below run **inside the WSL2 Ubuntu environment**, not a separate VirtualBox VM
- Supported Ubuntu releases: 24.04 LTS, 22.04 LTS, or newer

## 1. Install Docker Engine

```bash
# Remove any conflicting packages
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do
  sudo apt-get remove $pkg
done

# Add Docker's GPG key
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the Docker repository
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update

# Install Docker
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo usermod -aG docker $USER && newgrp docker

# Verify
docker version
sudo systemctl status docker
sudo systemctl start docker   # if not already running
sudo docker run hello-world   # smoke test
```

## 2. Install Minikube

```bash
sudo apt update -y && sudo apt upgrade -y
sudo apt install -y curl conntrack
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
minikube version
```

## 3. Install kubectl

```bash
# Method 1 (preferred)
sudo snap install kubectl --classic

# Method 2 (if snap install fails) - download in Windows, then move into WSL
# Download from: https://dl.k8s.io/release/<version>/bin/linux/amd64/kubectl
cd <path_to_downloaded_kubectl>   # e.g. /mnt/c/Users/<you>/Downloads
chmod +x kubectl
sudo mv kubectl /snap/bin
export PATH=$PATH:/snap/bin       # if /snap/bin isn't already on PATH

kubectl version --client
```

## 4. Start the Minikube Cluster

```bash
# Docker Engine must be running first
minikube start --driver=docker
minikube status
kubectl get nodes
```

## 5. Deploy Postgres and OpenNMS

See [`k8s/secret.yaml`](../k8s/secret.yaml), [`k8s/postgres-deployment.yaml`](../k8s/postgres-deployment.yaml), and [`k8s/opennms-deployment.yaml`](../k8s/opennms-deployment.yaml) for the manifests, and the Quick Start section in the main [README](../README.md) for the apply order.

```bash
kubectl get svc    # confirm postgres:5432 and opennms:8980 are running
kubectl get pods   # confirm pods are Running/Ready
```

## 6. Access Postgres from Windows (optional)

```bash
kubectl port-forward svc/postgres 5432:5432
```

Then configure a DB client (e.g. pgAdmin, DBeaver) on Windows with:
- Host: `localhost`
- Port: `5432`
- Database / User: `opennms`

## 7. Access OpenNMS

```bash
kubectl port-forward svc/opennms 8980:8980
```

Open `http://localhost:8980/opennms/` in a browser (default login: `admin` / `admin` — change this if running anything beyond a local lab).

## 8. Validate: Provision a Test Node

Inside the OpenNMS UI:

1. **Admin → Provisioning Requisitions → Add Requisition**
2. Name it `test-requisition`
3. Add a node, e.g. `test-node`
4. Under the **Interfaces** tab, add an interface with IP `127.0.0.1` and save
5. Click **Synchronize**
6. Confirm the **"Nodes in Database"** count changes from `0` to `1`
7. Verify under **Inventory** that `test-node (127.0.0.1)` appears

This confirms the full stack — Kubernetes networking, Postgres connectivity, and OpenNMS provisioning — is working end-to-end.
