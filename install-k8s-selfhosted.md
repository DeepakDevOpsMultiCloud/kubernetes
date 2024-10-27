# Installing Kubernetes on a Self-Hosted Agent

This guide provides step-by-step instructions to install Kubernetes (K8s) on a self-hosted agent, including Docker installation and cluster initialization.

## Step 1: Pass User Data for Initial Setup

Create a user data script that installs Docker and Kubernetes components. This script can be passed during the VM instance creation.

```bash
#!/bin/bash
# Update package index
apt-get update

# Install Docker
curl -fsSL https://get.docker.com -o install-docker.sh
sh install-docker.sh

# Install cri-dockerd for Kubernetes
cd /tmp
wget https://github.com/Mirantis/cri-dockerd/releases/download/v0.3.15/cri-dockerd_0.3.15.3-0.ubuntu-jammy_amd64.deb
sudo dpkg -i cri-dockerd_0.3.15.3-0.ubuntu-jammy_amd64.deb

# Install necessary packages for Kubernetes
apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gpg

# Add Kubernetes APT repository
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.31/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.31/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

# Install Kubernetes components
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl

# Hold the versions of installed packages
sudo apt-mark hold kubelet kubeadm kubectl

# Enable kubelet service
sudo systemctl enable --now kubelet
```

### Step 2: Initialize the Kubernetes Cluster
After the VM is set up and the above script has run, log into the master node and initialize the cluster:
```bash
kubeadm init --pod-network-cidr=10.244.0.0/16 --cri-socket "unix:///var/run/cri-dockerd.sock"
```
### Step 3: Set Up User Permissions
As a normal user (non-root), execute the following commands to create the necessary directories as indicated by the kubeadm init output:
```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
### Step 4: Join Worker Nodes to the Cluster
Log into each worker node and execute the join command provided at the end of the kubeadm init output:
```bash
kubeadm join <master-node-ip>:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash> --cri-socket "unix:///var/run/cri-dockerd.sock"
```
### Step 5: Install Pod Network Add-on
To fix any "Not Ready" status for nodes, install a pod network add-on such as Flannel on the master node:
```bash
kubectl apply -f https://github.com/coreos/flannel/raw/master/Documentation/kube-flannel.yml
```
### Step 6: Enable Command Auto-completion (Optional)
For command auto-completion in bash, run:
```bash
source <(kubectl completion bash) # Set up autocomplete in current shell.
echo "source <(kubectl completion bash)" >> ~/.bashrc # Add autocomplete permanently.
```
Additional Documentation
For more details on Kubernetes API or other configurations, refer to the official documentation:
[Kubernetes API Reference ](https://kubernetes.io/docs/reference/)
