# Minikube Installation and Setup on Ubuntu

## Overview

This guide explains how to install and configure **Minikube on Ubuntu Linux** using Docker as the driver. Minikube runs a local Kubernetes cluster for learning, development, testing, and practicing Kubernetes deployments.

## Table of Contents

- [Prerequisites](#prerequisites)
- [Step 1: Update Ubuntu](#step-1-update-ubuntu)
- [Step 2: Install Docker](#step-2-install-docker)
- [Step 3: Configure Docker](#step-3-configure-docker)
- [Step 4: Install kubectl](#step-4-install-kubectl)
- [Step 5: Install Minikube](#step-5-install-minikube)
- [Step 6: Start the Cluster](#step-6-start-the-cluster)
- [Step 7: Verify the Installation](#step-7-verify-the-installation)
- [Step 8: Deploy a Sample Application](#step-8-deploy-a-sample-application)
- [Useful Commands](#useful-commands)
- [Troubleshooting](#troubleshooting)
- [References](#references)

## Prerequisites

Recommended minimum system requirements:

- Ubuntu Linux
- 2 CPUs or more
- 2 GB of available memory
- 20 GB of free disk space
- Internet connectivity
- Sudo privileges

**Tools required:**
- Docker — container runtime and Minikube driver
- Minikube — local Kubernetes cluster
- kubectl — Kubernetes command-line tool

## Step 1: Update Ubuntu

Update the package index and installed packages.

```bash
sudo apt update
sudo apt upgrade -y
```

Install required utilities:

```bash
sudo apt install -y curl wget ca-certificates gnupg apt-transport-https
```

## Step 2: Install Docker

Configure Docker's official APT repository.

```bash
sudo install -m 0755 -d /etc/apt/keyrings

sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
  -o /etc/apt/keyrings/docker.asc

sudo chmod a+r /etc/apt/keyrings/docker.asc
```

Add the Docker repository:

```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" \
  | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

Update the package index:

```bash
sudo apt update
```

Install Docker Engine and its plugins:

```bash
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

Start Docker and enable it at boot:

```bash
sudo systemctl enable --now docker
```

Verify Docker:

```bash
sudo systemctl status docker
sudo docker run hello-world
```

## Step 3: Configure Docker

Allow your current user to run Docker commands without `sudo`.

```bash
sudo usermod -aG docker "$USER"
```

Apply the group membership by logging out and logging back in, or run:

```bash
newgrp docker
```

Verify access:

```bash
docker run hello-world
docker version
```

**Security note:** Membership in the Docker group grants root-level privileges on the host. Only add trusted users to this group.

## Step 4: Install kubectl

Download the latest stable kubectl release for your system architecture.

```bash
ARCH=$(dpkg --print-architecture)

curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/${ARCH}/kubectl"

sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

rm kubectl
```

Verify the installation:

```bash
kubectl version --client
```

## Step 5: Install Minikube

Download the latest stable Minikube binary for your architecture.

For standard Ubuntu `amd64` or `x86_64` systems:

```bash
curl -LO https://github.com/kubernetes/minikube/releases/latest/download/minikube-linux-amd64

sudo install minikube-linux-amd64 /usr/local/bin/minikube

rm minikube-linux-amd64
```

For ARM64 systems, replace `amd64` in the download filename with `arm64`.

Verify Minikube:

```bash
minikube version
```

## Step 6: Start the Cluster

Configure Docker as the default driver:

```bash
minikube config set driver docker
```

Start the Kubernetes cluster:

```bash
minikube start --driver=docker
```

Minikube will download the required Kubernetes components and create a local cluster. The initial startup may take several minutes depending on your internet connection and system resources.

## Step 7: Verify the Installation

Check the cluster status:

```bash
minikube status
```

Check Kubernetes nodes:

```bash
kubectl get nodes
```

Check system pods:

```bash
kubectl get pods -A
```

Check cluster information:

```bash
kubectl cluster-info
```

The node should eventually show `Ready` status, and the core Kubernetes components should become healthy.

## Step 8: Deploy a Sample Application

Deploy an NGINX web server to verify that Kubernetes is working.

Create a Deployment:

```bash
kubectl create deployment nginx --image=nginx
```

Expose it through a Kubernetes Service:

```bash
kubectl expose deployment nginx \
  --type=NodePort \
  --port=80
```

Check the Deployment and Service:

```bash
kubectl get deployments
kubectl get pods
kubectl get services
```

Wait until the NGINX pod is running:

```bash
kubectl rollout status deployment/nginx
```

Open the application in your browser:

```bash
minikube service nginx
```

Alternatively, retrieve the service URL:

```bash
minikube service nginx --url
```

On Linux, the Docker driver may require the service command to remain running to provide access to the service URL. If the URL is not reachable directly, use `minikube service nginx` in a terminal and keep that terminal open.

Clean up the test resources when finished:

```bash
kubectl delete service nginx
kubectl delete deployment nginx
```

## Useful Commands

| Command | Description |
|---|---|
| `minikube start` | Start the cluster |
| `minikube stop` | Stop the cluster |
| `minikube status` | Check cluster status |
| `minikube delete` | Delete the cluster |
| `minikube dashboard` | Open the Kubernetes Dashboard |
| `minikube addons list` | List available addons |
| `kubectl get nodes` | List Kubernetes nodes |
| `kubectl get pods -A` | List pods across namespaces |
| `kubectl get services -A` | List services across namespaces |
| `kubectl describe pod <pod-name>` | Inspect a pod |
| `kubectl logs <pod-name>` | View pod logs |

## Troubleshooting

### 1. Docker permission denied

**Problem:** Docker commands fail with a permission error.

**Solution:**

```bash
sudo usermod -aG docker "$USER"
```

Log out and log back in, then verify:

```bash
docker ps
```

### 2. Minikube fails to start

Check Docker:

```bash
sudo systemctl status docker
docker info
```

Check Minikube logs:

```bash
minikube logs
```

Try restarting the cluster:

```bash
minikube stop
minikube start --driver=docker
```

### 3. kubectl cannot connect to the cluster

Check the cluster:

```bash
minikube status
kubectl config current-context
```

If necessary, start Minikube again:

```bash
minikube start --driver=docker
```

### 4. Insufficient resources

Close unnecessary applications and check available resources:

```bash
free -h
df -h
nproc
```

Ensure the system has sufficient RAM, CPU, and disk space for the cluster.

### 5. Reset the cluster

**Warning:** This deletes the local Minikube cluster and its Kubernetes resources.

```bash
minikube delete
minikube start --driver=docker
```

## References

- Minikube official documentation: https://minikube.sigs.k8s.io/docs/start/
- Minikube Docker driver: https://minikube.sigs.k8s.io/docs/drivers/docker/
- Kubernetes kubectl installation: https://kubernetes.io/docs/tasks/tools/
- Docker Engine on Ubuntu: https://docs.docker.com/engine/install/ubuntu/

## Conclusion

Minikube is now installed and configured on Ubuntu with Docker as the driver. You can use this environment to practice Kubernetes Deployments, Services, ConfigMaps, Secrets, Namespaces, Ingress, and Helm.

**Note:** Minikube is intended for local development and learning, not as a production Kubernetes cluster.
