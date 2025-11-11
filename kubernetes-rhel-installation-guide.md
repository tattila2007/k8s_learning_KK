# Complete Kubernetes Cluster Installation Guide for RHEL 9 & 10

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Infrastructure Planning](#infrastructure-planning)
3. [Node Preparation](#node-preparation)
4. [Container Runtime Installation](#container-runtime-installation)
5. [Kubernetes Installation](#kubernetes-installation)
6. [Control Plane Initialization](#control-plane-initialization)
7. [Worker Node Configuration](#worker-node-configuration)
8. [Network Plugin Installation](#network-plugin-installation)
9. [Verification and Testing](#verification-and-testing)
10. [Troubleshooting](#troubleshooting)

---

## Prerequisites

### Minimum Hardware Requirements

**Control Plane Node(s):**
- 2 CPUs or more
- 2 GB RAM minimum (4 GB recommended)
- 20 GB disk space
- Network connectivity between all nodes

**Worker Node(s):**
- 1 CPU or more
- 2 GB RAM minimum (4 GB recommended)
- 20 GB disk space
- Network connectivity between all nodes

### Software Requirements
- RHEL 9.x or 10.x with valid subscription
- Root or sudo access on all nodes
- Unique hostname, MAC address, and product_uuid for each node

---

## Infrastructure Planning

### Network Planning

1. **Pod Network CIDR**: Default `10.244.0.0/16` (can be customized)
2. **Service CIDR**: Default `10.96.0.0/12` (can be customized)
3. **Required Ports**:

**Control Plane Node:**
- TCP 6443: Kubernetes API server
- TCP 2379-2380: etcd server client API
- TCP 10250: Kubelet API
- TCP 10259: kube-scheduler
- TCP 10257: kube-controller-manager

**Worker Nodes:**
- TCP 10250: Kubelet API
- TCP 30000-32767: NodePort Services

### Cluster Architecture Example

```
┌─────────────────────────────────────────────┐
│         Control Plane Node                  │
│  (master.example.com - 192.168.1.10)       │
└─────────────────────────────────────────────┘
                    │
        ┌───────────┴───────────┐
        │                       │
┌───────▼──────┐       ┌───────▼──────┐
│ Worker Node 1│       │ Worker Node 2│
│ 192.168.1.11 │       │ 192.168.1.12 │
└──────────────┘       └──────────────┘
```

---

## Node Preparation

### Step 1: Update System and Set Hostname

Run on **ALL nodes**:

```bash
# Update system
sudo dnf update -y

# Set hostname (change accordingly for each node)
# Control plane node:
sudo hostnamectl set-hostname master.example.com

# Worker nodes:
sudo hostnamectl set-hostname worker1.example.com
sudo hostnamectl set-hostname worker2.example.com

# Verify
hostnamectl
```

### Step 2: Configure /etc/hosts

Add entries on **ALL nodes**:

```bash
sudo tee -a /etc/hosts <<EOF
192.168.1.10  master.example.com  master
192.168.1.11  worker1.example.com worker1
192.168.1.12  worker2.example.com worker2
EOF
```

### Step 3: Disable SELinux

```bash
# Set to permissive mode
sudo setenforce 0
sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

# Verify
getenforce
```

**Note**: For production, consider configuring SELinux policies instead of disabling it.

### Step 4: Disable Swap

Kubernetes requires swap to be disabled:

```bash
# Disable swap immediately
sudo swapoff -a

# Disable swap permanently
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

# Verify
free -h
swapon --show
```

### Step 5: Configure Firewall

**Option A: Disable firewall (for testing/lab environments)**

```bash
sudo systemctl stop firewalld
sudo systemctl disable firewalld
```

**Option B: Configure firewall rules (recommended for production)**

**On Control Plane Node:**

```bash
sudo firewall-cmd --permanent --add-port=6443/tcp
sudo firewall-cmd --permanent --add-port=2379-2380/tcp
sudo firewall-cmd --permanent --add-port=10250/tcp
sudo firewall-cmd --permanent --add-port=10259/tcp
sudo firewall-cmd --permanent --add-port=10257/tcp
sudo firewall-cmd --reload
```

**On Worker Nodes:**

```bash
sudo firewall-cmd --permanent --add-port=10250/tcp
sudo firewall-cmd --permanent --add-port=30000-32767/tcp
sudo firewall-cmd --reload
```

### Step 6: Load Kernel Modules

```bash
# Create configuration file
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

# Load modules
sudo modprobe overlay
sudo modprobe br_netfilter

# Verify
lsmod | grep overlay
lsmod | grep br_netfilter
```

### Step 7: Configure Sysctl Parameters

```bash
# Create sysctl configuration
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

# Apply settings
sudo sysctl --system

# Verify
sysctl net.bridge.bridge-nf-call-iptables net.bridge.bridge-nf-call-ip6tables net.ipv4.ip_forward
```

---

## Container Runtime Installation

Kubernetes requires a container runtime. We'll use **containerd**.

### Step 1: Add Docker Repository

```bash
# Install required packages
sudo dnf install -y dnf-utils device-mapper-persistent-data lvm2

# Add Docker repository
sudo dnf config-manager --add-repo https://download.docker.com/linux/rhel/docker-ce.repo
```

### Step 2: Install containerd

```bash
# Install containerd
sudo dnf install -y containerd.io

# Create default configuration directory
sudo mkdir -p /etc/containerd

# Generate default configuration
sudo containerd config default | sudo tee /etc/containerd/config.toml
```

### Step 3: Configure containerd to Use systemd cgroup Driver

```bash
# Edit containerd configuration to use systemd cgroup driver
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml

# Verify the change
grep SystemdCgroup /etc/containerd/config.toml
```

### Step 4: Start and Enable containerd

```bash
# Restart containerd
sudo systemctl restart containerd

# Enable containerd to start on boot
sudo systemctl enable containerd

# Verify status
sudo systemctl status containerd
```

---

## Kubernetes Installation

### Step 1: Add Kubernetes Repository

```bash
# Add Kubernetes repository
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://pkgs.k8s.io/core:/stable:/v1.31/rpm/
enabled=1
gpgcheck=1
gpgkey=https://pkgs.k8s.io/core:/stable:/v1.31/rpm/repodata/repomd.xml.key
exclude=kubelet kubeadm kubectl cri-tools kubernetes-cni
EOF
```

**Note**: Replace `v1.31` with your desired Kubernetes version (e.g., `v1.30`, `v1.29`).

### Step 2: Install Kubernetes Components

```bash
# Install kubelet, kubeadm, and kubectl
sudo dnf install -y kubelet kubeadm kubectl --disableexcludes=kubernetes

# Enable kubelet service
sudo systemctl enable --now kubelet
```

### Step 3: Verify Installation

```bash
# Check versions
kubeadm version
kubelet --version
kubectl version --client

# Check kubelet status (it will fail until cluster is initialized)
sudo systemctl status kubelet
```

---

## Control Plane Initialization

**Run these steps ONLY on the Control Plane Node.**

### Step 1: Initialize the Control Plane

```bash
# Initialize with default settings
sudo kubeadm init --pod-network-cidr=10.244.0.0/16

# OR with custom settings (example)
sudo kubeadm init \
  --pod-network-cidr=10.244.0.0/16 \
  --apiserver-advertise-address=192.168.1.10 \
  --control-plane-endpoint=master.example.com \
  --upload-certs
```

**Important**: Save the output! It contains the join command for worker nodes.

Example output:
```
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join master.example.com:6443 --token abcdef.0123456789abcdef \
    --discovery-token-ca-cert-hash sha256:1234567890abcdef...
```

### Step 2: Configure kubectl

```bash
# For regular user
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# For root user (if needed)
export KUBECONFIG=/etc/kubernetes/admin.conf
echo 'export KUBECONFIG=/etc/kubernetes/admin.conf' >> /root/.bashrc
```

### Step 3: Verify Control Plane

```bash
# Check nodes
kubectl get nodes

# Check pods in all namespaces
kubectl get pods -A

# Check cluster info
kubectl cluster-info
```

---

## Worker Node Configuration

**Run these steps on each Worker Node.**

### Step 1: Join Worker Nodes to Cluster

Use the `kubeadm join` command from the control plane initialization output:

```bash
sudo kubeadm join master.example.com:6443 \
  --token abcdef.0123456789abcdef \
  --discovery-token-ca-cert-hash sha256:1234567890abcdef...
```

### Step 2: Verify Worker Node Joined (from Control Plane)

```bash
# Check nodes
kubectl get nodes

# Check in detail
kubectl get nodes -o wide
```

**Note**: Nodes will show as `NotReady` until the network plugin is installed.

### Step 3: Generate New Join Token (if needed)

If the original token expired or you need a new one:

```bash
# On control plane node
kubeadm token create --print-join-command
```

---

## Network Plugin Installation

A network plugin is required for pod networking. We'll use **Calico** or **Flannel**.

### Option A: Install Calico

```bash
# Download Calico manifest
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.27.0/manifests/tigera-operator.yaml

# Download custom resources
curl https://raw.githubusercontent.com/projectcalico/calico/v3.27.0/manifests/custom-resources.yaml -O

# Edit if you used a different pod CIDR (default is 192.168.0.0/16)
# Change it to match your pod-network-cidr (e.g., 10.244.0.0/16)
sed -i 's|192.168.0.0/16|10.244.0.0/16|g' custom-resources.yaml

# Apply custom resources
kubectl create -f custom-resources.yaml

# Verify Calico pods
kubectl get pods -n calico-system
```

### Option B: Install Flannel

```bash
# Apply Flannel manifest
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml

# Verify Flannel pods
kubectl get pods -n kube-flannel
```

### Step 2: Verify Network Plugin

```bash
# Check all pods are running
kubectl get pods -A

# Check nodes are Ready
kubectl get nodes
```

Wait until all nodes show `Ready` status.

---

## Verification and Testing

### Step 1: Deploy Test Application

```bash
# Create nginx deployment
kubectl create deployment nginx --image=nginx

# Expose the deployment
kubectl expose deployment nginx --port=80 --type=NodePort

# Check deployment
kubectl get deployments
kubectl get pods
kubectl get services
```

### Step 2: Test Pod-to-Pod Communication

```bash
# Create a test pod
kubectl run test-pod --image=busybox --restart=Never -- sleep 3600

# Get nginx pod IP
NGINX_POD_IP=$(kubectl get pod -l app=nginx -o jsonpath='{.items[0].status.podIP}')

# Test connectivity from test pod
kubectl exec test-pod -- wget -qO- http://$NGINX_POD_IP
```

### Step 3: Test DNS Resolution

```bash
# Test DNS from test pod
kubectl exec test-pod -- nslookup kubernetes.default

# Clean up
kubectl delete pod test-pod
```

### Step 4: Check Cluster Health

```bash
# Check component status
kubectl get componentstatuses

# Check cluster info
kubectl cluster-info

# Check all system pods
kubectl get pods -n kube-system

# Check nodes
kubectl get nodes -o wide
```

---

## Troubleshooting

### Common Issues and Solutions

#### 1. Nodes Show "NotReady" Status

**Check:**
```bash
# Check node details
kubectl describe node <node-name>

# Check kubelet logs
sudo journalctl -u kubelet -f

# Check CNI plugin
kubectl get pods -n kube-system
kubectl get pods -n calico-system  # or kube-flannel
```

**Solution:** Ensure network plugin is properly installed and all pods are running.

#### 2. Kubelet Not Starting

**Check:**
```bash
# Check kubelet status
sudo systemctl status kubelet

# View detailed logs
sudo journalctl -xeu kubelet

# Check if swap is disabled
free -h
swapon --show
```

**Solution:** Ensure swap is disabled and container runtime is running.

#### 3. Pods Stuck in "Pending" State

**Check:**
```bash
# Describe the pod
kubectl describe pod <pod-name>

# Check node resources
kubectl describe nodes
```

**Solution:** Check for resource constraints or node taints.

#### 4. Unable to Connect to API Server

**Check:**
```bash
# Check API server pod
kubectl get pods -n kube-system | grep apiserver

# Check certificates
sudo kubeadm certs check-expiration

# Verify connectivity
curl -k https://localhost:6443/healthz
```

**Solution:** Verify firewall rules and certificate validity.

#### 5. Container Runtime Issues

**Check:**
```bash
# Check containerd status
sudo systemctl status containerd

# Check containerd logs
sudo journalctl -u containerd -f

# Test containerd
sudo ctr version
```

**Solution:** Restart containerd and verify configuration.

### Useful Debugging Commands

```bash
# View cluster events
kubectl get events -A --sort-by='.lastTimestamp'

# Check kubelet configuration
sudo cat /var/lib/kubelet/config.yaml

# View kubeadm configuration
kubectl get cm kubeadm-config -n kube-system -o yaml

# Reset node (WARNING: removes all Kubernetes components)
sudo kubeadm reset
```

### Log Locations

- **Kubelet logs**: `journalctl -u kubelet`
- **Containerd logs**: `journalctl -u containerd`
- **Pod logs**: `kubectl logs <pod-name> -n <namespace>`
- **Kubernetes manifests**: `/etc/kubernetes/manifests/`
- **Kubelet config**: `/var/lib/kubelet/config.yaml`

---

## Additional Configurations

### Enable kubectl Autocompletion

```bash
# For bash
kubectl completion bash | sudo tee /etc/bash_completion.d/kubectl
source ~/.bashrc

# For current session
source <(kubectl completion bash)

# Add alias
echo 'alias k=kubectl' >> ~/.bashrc
echo 'complete -o default -F __start_kubectl k' >> ~/.bashrc
source ~/.bashrc
```

### Install Kubernetes Dashboard (Optional)

```bash
# Install dashboard
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml

# Create admin user
cat <<EOF | kubectl apply -f -
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
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard
EOF

# Get token
kubectl -n kubernetes-dashboard create token admin-user

# Access dashboard (use token from above)
kubectl proxy
# Access at: http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/
```

### Install Metrics Server (Optional)

```bash
# Install metrics server
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

# Verify
kubectl get deployment metrics-server -n kube-system
kubectl top nodes
```

---

## Cluster Maintenance

### Upgrading the Cluster

**Control Plane:**
```bash
# Check available versions
sudo dnf list --showduplicates kubeadm --disableexcludes=kubernetes

# Upgrade kubeadm
sudo dnf install -y kubeadm-1.31.x-0 --disableexcludes=kubernetes

# Verify upgrade plan
sudo kubeadm upgrade plan

# Apply upgrade
sudo kubeadm upgrade apply v1.31.x

# Upgrade kubelet and kubectl
sudo dnf install -y kubelet-1.31.x-0 kubectl-1.31.x-0 --disableexcludes=kubernetes
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```

**Worker Nodes:**
```bash
# Upgrade kubeadm
sudo dnf install -y kubeadm-1.31.x-0 --disableexcludes=kubernetes

# Upgrade node
sudo kubeadm upgrade node

# Upgrade kubelet and kubectl
sudo dnf install -y kubelet-1.31.x-0 kubectl-1.31.x-0 --disableexcludes=kubernetes
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```

### Backing Up etcd

```bash
# Backup etcd (on control plane)
sudo ETCDCTL_API=3 etcdctl snapshot save /tmp/etcd-backup.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key

# Verify backup
sudo ETCDCTL_API=3 etcdctl snapshot status /tmp/etcd-backup.db
```

---

## Security Best Practices

1. **Keep SELinux enabled** in production with proper policies
2. **Use RBAC** for access control
3. **Enable Pod Security Standards**
4. **Rotate certificates** regularly
5. **Use network policies** to restrict pod communication
6. **Keep cluster updated** with latest security patches
7. **Use secrets** for sensitive data
8. **Enable audit logging**
9. **Restrict API server access**
10. **Use private registry** for container images

---

## Summary

You now have a complete Kubernetes cluster running on RHEL 9/10. The cluster includes:

- ✅ Control plane node with all Kubernetes components
- ✅ Worker nodes joined to the cluster
- ✅ Container runtime (containerd) configured
- ✅ Network plugin for pod communication
- ✅ Basic security configurations

**Next Steps:**
1. Deploy your applications
2. Set up monitoring and logging
3. Configure persistent storage
4. Implement backup strategies
5. Set up ingress controller for external access

**Useful Resources:**
- Official Kubernetes Documentation: https://kubernetes.io/docs/
- Kubernetes on RHEL: https://access.redhat.com/documentation/
- kubectl Cheat Sheet: https://kubernetes.io/docs/reference/kubectl/cheatsheet/

---

**Document Version**: 1.0  
**Last Updated**: November 2025  
**Compatible with**: RHEL 9.x, RHEL 10.x, Kubernetes 1.28+
