# Kubernetes Offline Installation - Image Download Guide

## Table of Contents
1. [Overview](#overview)
2. [Prerequisites](#prerequisites)
3. [Identify Required Images](#identify-required-images)
4. [Download Container Images](#download-container-images)
5. [Download RPM Packages](#download-rpm-packages)
6. [Transfer to Offline Environment](#transfer-to-offline-environment)
7. [Load Images in Offline Environment](#load-images-in-offline-environment)
8. [Complete Offline Installation Scripts](#complete-offline-installation-scripts)

---

## Overview

For offline Kubernetes installation, you need to download:
1. **Container images** (for Kubernetes components and CNI plugin)
2. **RPM packages** (for containerd, kubelet, kubeadm, kubectl)
3. **Network plugin manifests** (Calico or Flannel)

---

## Prerequisites

### Online Machine Requirements
- Internet connection
- Docker or containerd installed
- RHEL 9/10 (same version as target offline machines)
- Sufficient disk space (at least 10 GB free)

### Tools Installation on Online Machine

```bash
# Install required tools
sudo dnf install -y yum-utils createrepo

# Install skopeo for image operations (optional but recommended)
sudo dnf install -y skopeo

# Verify Docker or containerd is running
sudo systemctl status docker
# OR
sudo systemctl status containerd
```

---

## Identify Required Images

### Method 1: Using kubeadm to List Images

```bash
# First, install kubeadm on the online machine
sudo dnf install -y kubeadm

# List required images for specific Kubernetes version
kubeadm config images list --kubernetes-version=v1.31.0

# Example output:
# registry.k8s.io/kube-apiserver:v1.31.0
# registry.k8s.io/kube-controller-manager:v1.31.0
# registry.k8s.io/kube-scheduler:v1.31.0
# registry.k8s.io/kube-proxy:v1.31.0
# registry.k8s.io/coredns/coredns:v1.11.1
# registry.k8s.io/pause:3.10
# registry.k8s.io/etcd:3.5.15-0
```

### Method 2: Manual List of All Required Images

Create a file with all required images:

```bash
cat > k8s-images-list.txt <<EOF
# Kubernetes Core Components (v1.31.0)
registry.k8s.io/kube-apiserver:v1.31.0
registry.k8s.io/kube-controller-manager:v1.31.0
registry.k8s.io/kube-scheduler:v1.31.0
registry.k8s.io/kube-proxy:v1.31.0
registry.k8s.io/coredns/coredns:v1.11.1
registry.k8s.io/pause:3.10
registry.k8s.io/etcd:3.5.15-0

# Calico Network Plugin (v3.27.0) - Choose if using Calico
docker.io/calico/cni:v3.27.0
docker.io/calico/node:v3.27.0
docker.io/calico/kube-controllers:v3.27.0
quay.io/tigera/operator:v1.32.5
docker.io/calico/typha:v3.27.0
docker.io/calico/pod2daemon-flexvol:v3.27.0

# Flannel Network Plugin - Choose if using Flannel
docker.io/flannel/flannel:v0.24.0
docker.io/flannel/flannel-cni-plugin:v1.2.0

# Metrics Server (optional but recommended)
registry.k8s.io/metrics-server/metrics-server:v0.7.0
EOF
```

**Note**: Adjust versions based on your requirements.

---

## Download Container Images

### Method 1: Using Docker (Recommended)

```bash
#!/bin/bash
# Script: download-k8s-images-docker.sh

# Create directory for images
mkdir -p ~/k8s-offline/images
cd ~/k8s-offline/images

# Read images from file and download
while IFS= read -r image; do
    # Skip empty lines and comments
    [[ -z "$image" || "$image" =~ ^#.*$ ]] && continue
    
    echo "Pulling image: $image"
    docker pull "$image"
    
    # Save image to tar file
    image_name=$(echo "$image" | tr '/:' '_')
    echo "Saving image to: ${image_name}.tar"
    docker save "$image" -o "${image_name}.tar"
    
done < ../k8s-images-list.txt

echo "All images downloaded and saved!"
ls -lh
```

Make it executable and run:

```bash
chmod +x download-k8s-images-docker.sh
./download-k8s-images-docker.sh
```

### Method 2: Using containerd and ctr

```bash
#!/bin/bash
# Script: download-k8s-images-ctr.sh

mkdir -p ~/k8s-offline/images
cd ~/k8s-offline/images

while IFS= read -r image; do
    [[ -z "$image" || "$image" =~ ^#.*$ ]] && continue
    
    echo "Pulling image: $image"
    sudo ctr image pull "$image"
    
    image_name=$(echo "$image" | tr '/:' '_')
    echo "Exporting image to: ${image_name}.tar"
    sudo ctr image export "${image_name}.tar" "$image"
    
done < ../k8s-images-list.txt

echo "All images downloaded and saved!"
```

### Method 3: Using skopeo (More Flexible)

```bash
#!/bin/bash
# Script: download-k8s-images-skopeo.sh

mkdir -p ~/k8s-offline/images
cd ~/k8s-offline/images

while IFS= read -r image; do
    [[ -z "$image" || "$image" =~ ^#.*$ ]] && continue
    
    echo "Copying image: $image"
    image_name=$(echo "$image" | tr '/:' '_')
    
    skopeo copy \
        docker://"$image" \
        docker-archive:"${image_name}.tar:$image"
    
done < ../k8s-images-list.txt

echo "All images downloaded!"
```

### Method 4: Download All Images at Once (Single Archive)

```bash
#!/bin/bash
# Script: download-k8s-images-single-archive.sh

mkdir -p ~/k8s-offline
cd ~/k8s-offline

echo "Pulling all images..."

# Pull all images first
while IFS= read -r image; do
    [[ -z "$image" || "$image" =~ ^#.*$ ]] && continue
    echo "Pulling: $image"
    docker pull "$image"
done < k8s-images-list.txt

echo "Creating single archive with all images..."
docker save $(grep -v '^#' k8s-images-list.txt | grep -v '^$' | tr '\n' ' ') \
    -o k8s-images-all.tar

echo "Archive created: k8s-images-all.tar"
ls -lh k8s-images-all.tar
```

---

## Download RPM Packages

### Method 1: Download Specific Packages with Dependencies

```bash
#!/bin/bash
# Script: download-k8s-rpms.sh

# Create directory for RPMs
mkdir -p ~/k8s-offline/rpms
cd ~/k8s-offline/rpms

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

# Add Docker repository for containerd
sudo dnf config-manager --add-repo https://download.docker.com/linux/rhel/docker-ce.repo

# Download Kubernetes packages
echo "Downloading Kubernetes packages..."
sudo dnf download --resolve --alldeps --disableexcludes=kubernetes \
    kubelet \
    kubeadm \
    kubectl \
    kubernetes-cni \
    cri-tools

# Download containerd
echo "Downloading containerd..."
sudo dnf download --resolve --alldeps containerd.io

# Download additional dependencies
echo "Downloading additional dependencies..."
sudo dnf download --resolve --alldeps \
    socat \
    conntrack-tools \
    ebtables \
    ethtool \
    iptables \
    iproute-tc

echo "All RPMs downloaded!"
ls -lh
```

### Method 2: Create Local Repository

```bash
#!/bin/bash
# Script: create-local-repo.sh

mkdir -p ~/k8s-offline/local-repo
cd ~/k8s-offline/local-repo

# Download all packages
sudo dnf download --resolve --alldeps --disableexcludes=kubernetes \
    kubelet kubeadm kubectl kubernetes-cni cri-tools containerd.io \
    socat conntrack-tools ebtables ethtool iptables iproute-tc

# Create repository metadata
createrepo .

echo "Local repository created!"
echo "Repository location: $(pwd)"
```

### Method 3: Download Specific Versions

```bash
#!/bin/bash
# Script: download-specific-k8s-versions.sh

K8S_VERSION="1.31.0"
mkdir -p ~/k8s-offline/rpms-${K8S_VERSION}
cd ~/k8s-offline/rpms-${K8S_VERSION}

# Download specific versions
sudo dnf download --resolve --alldeps --disableexcludes=kubernetes \
    kubelet-${K8S_VERSION}-* \
    kubeadm-${K8S_VERSION}-* \
    kubectl-${K8S_VERSION}-*

# Download latest containerd and dependencies
sudo dnf download --resolve --alldeps \
    containerd.io \
    kubernetes-cni \
    cri-tools

echo "Specific version packages downloaded!"
```

---

## Transfer to Offline Environment

### Option 1: Create Complete Archive

```bash
#!/bin/bash
# Script: create-offline-bundle.sh

echo "Creating offline installation bundle..."

cd ~
mkdir -p k8s-offline-bundle

# Copy images
cp -r k8s-offline/images k8s-offline-bundle/
cp -r k8s-offline/rpms k8s-offline-bundle/

# Download CNI plugin manifests
mkdir -p k8s-offline-bundle/manifests

# Calico manifests
curl -o k8s-offline-bundle/manifests/tigera-operator.yaml \
    https://raw.githubusercontent.com/projectcalico/calico/v3.27.0/manifests/tigera-operator.yaml

curl -o k8s-offline-bundle/manifests/custom-resources.yaml \
    https://raw.githubusercontent.com/projectcalico/calico/v3.27.0/manifests/custom-resources.yaml

# Flannel manifest (alternative)
curl -o k8s-offline-bundle/manifests/kube-flannel.yml \
    https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml

# Metrics server manifest
curl -o k8s-offline-bundle/manifests/metrics-server.yaml \
    https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

# Create tarball
tar -czf k8s-offline-bundle.tar.gz k8s-offline-bundle/

echo "Bundle created: k8s-offline-bundle.tar.gz"
ls -lh k8s-offline-bundle.tar.gz
```

### Option 2: Transfer Individual Components

```bash
# Calculate sizes
du -sh ~/k8s-offline/images
du -sh ~/k8s-offline/rpms

# Create separate archives
cd ~/k8s-offline
tar -czf k8s-images.tar.gz images/
tar -czf k8s-rpms.tar.gz rpms/

# Transfer files to offline environment
# Using USB, SCP, or other method
scp k8s-images.tar.gz user@offline-server:/tmp/
scp k8s-rpms.tar.gz user@offline-server:/tmp/
```

---

## Load Images in Offline Environment

### On Offline Control Plane and Worker Nodes

#### Method 1: Load Individual Image Archives (Docker)

```bash
#!/bin/bash
# Script: load-k8s-images-docker.sh

cd /path/to/images

for tar_file in *.tar; do
    echo "Loading image from: $tar_file"
    docker load -i "$tar_file"
done

echo "All images loaded!"
docker images
```

#### Method 2: Load Single Archive (Docker)

```bash
# Load all images from single archive
docker load -i k8s-images-all.tar

# Verify
docker images
```

#### Method 3: Load Images with containerd

```bash
#!/bin/bash
# Script: load-k8s-images-containerd.sh

cd /path/to/images

for tar_file in *.tar; do
    echo "Importing image from: $tar_file"
    sudo ctr -n k8s.io image import "$tar_file"
done

echo "All images imported!"
sudo crictl images
```

#### Method 4: Load Single Archive (containerd)

```bash
# Import all images at once
sudo ctr -n k8s.io image import k8s-images-all.tar

# Verify
sudo crictl images
```

### Install RPM Packages

#### Method 1: Install from Directory

```bash
# Extract RPMs
cd /tmp
tar -xzf k8s-rpms.tar.gz

# Install all RPMs
cd rpms
sudo dnf localinstall -y *.rpm

# Verify installation
rpm -qa | grep kube
kubelet --version
kubeadm version
```

#### Method 2: Use Local Repository

```bash
# Extract repository
cd /tmp
tar -xzf k8s-rpms.tar.gz

# Create repository configuration
cat <<EOF | sudo tee /etc/yum.repos.d/k8s-local.repo
[kubernetes-local]
name=Kubernetes Local Repository
baseurl=file:///tmp/local-repo
enabled=1
gpgcheck=0
EOF

# Install packages
sudo dnf install -y kubelet kubeadm kubectl containerd.io

# Enable services
sudo systemctl enable --now kubelet
sudo systemctl enable --now containerd
```

---

## Complete Offline Installation Scripts

### Master Script: Complete Download Process

```bash
#!/bin/bash
# Script: complete-download.sh
# Purpose: Download everything needed for offline K8s installation

set -e

WORK_DIR="$HOME/k8s-offline"
K8S_VERSION="v1.31.0"
CALICO_VERSION="v3.27.0"
FLANNEL_VERSION="v0.24.0"

echo "=========================================="
echo "Kubernetes Offline Bundle Downloader"
echo "=========================================="
echo "K8s Version: $K8S_VERSION"
echo "Work Directory: $WORK_DIR"
echo ""

# Create directory structure
mkdir -p "$WORK_DIR"/{images,rpms,manifests,scripts}
cd "$WORK_DIR"

# Step 1: Create image list
echo "Step 1: Creating image list..."
cat > k8s-images-list.txt <<EOF
# Kubernetes Core Components
registry.k8s.io/kube-apiserver:v1.31.0
registry.k8s.io/kube-controller-manager:v1.31.0
registry.k8s.io/kube-scheduler:v1.31.0
registry.k8s.io/kube-proxy:v1.31.0
registry.k8s.io/coredns/coredns:v1.11.1
registry.k8s.io/pause:3.10
registry.k8s.io/etcd:3.5.15-0

# Calico Network Plugin
docker.io/calico/cni:v3.27.0
docker.io/calico/node:v3.27.0
docker.io/calico/kube-controllers:v3.27.0
quay.io/tigera/operator:v1.32.5
docker.io/calico/typha:v3.27.0
docker.io/calico/pod2daemon-flexvol:v3.27.0

# Flannel Network Plugin (alternative)
docker.io/flannel/flannel:v0.24.0
docker.io/flannel/flannel-cni-plugin:v1.2.0

# Metrics Server
registry.k8s.io/metrics-server/metrics-server:v0.7.0
EOF

# Step 2: Download images
echo "Step 2: Downloading container images..."
cd images

while IFS= read -r image; do
    [[ -z "$image" || "$image" =~ ^#.*$ ]] && continue
    
    echo "  Pulling: $image"
    docker pull "$image" || echo "  Warning: Failed to pull $image"
    
done < ../k8s-images-list.txt

echo "  Creating single image archive..."
docker save $(grep -v '^#' ../k8s-images-list.txt | grep -v '^$' | tr '\n' ' ') \
    -o k8s-images-all.tar

cd ..

# Step 3: Download RPMs
echo "Step 3: Downloading RPM packages..."

# Add repositories
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://pkgs.k8s.io/core:/stable:/v1.31/rpm/
enabled=1
gpgcheck=1
gpgkey=https://pkgs.k8s.io/core:/stable:/v1.31/rpm/repodata/repomd.xml.key
exclude=kubelet kubeadm kubectl cri-tools kubernetes-cni
EOF

sudo dnf config-manager --add-repo https://download.docker.com/linux/rhel/docker-ce.repo

cd rpms

sudo dnf download --resolve --alldeps --disableexcludes=kubernetes \
    kubelet kubeadm kubectl kubernetes-cni cri-tools containerd.io \
    socat conntrack-tools ebtables ethtool iptables iproute-tc \
    || echo "Warning: Some packages may not have downloaded"

cd ..

# Step 4: Download manifests
echo "Step 4: Downloading Kubernetes manifests..."
cd manifests

curl -sL -o tigera-operator.yaml \
    "https://raw.githubusercontent.com/projectcalico/calico/${CALICO_VERSION}/manifests/tigera-operator.yaml"

curl -sL -o calico-custom-resources.yaml \
    "https://raw.githubusercontent.com/projectcalico/calico/${CALICO_VERSION}/manifests/custom-resources.yaml"

curl -sL -o kube-flannel.yml \
    "https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml"

curl -sL -o metrics-server.yaml \
    "https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml"

cd ..

# Step 5: Create load scripts
echo "Step 5: Creating installation scripts..."

cat > scripts/01-load-images-docker.sh <<'SCRIPT'
#!/bin/bash
echo "Loading images with Docker..."
docker load -i ../images/k8s-images-all.tar
docker images
SCRIPT

cat > scripts/02-load-images-containerd.sh <<'SCRIPT'
#!/bin/bash
echo "Loading images with containerd..."
sudo ctr -n k8s.io image import ../images/k8s-images-all.tar
sudo crictl images
SCRIPT

cat > scripts/03-install-rpms.sh <<'SCRIPT'
#!/bin/bash
echo "Installing RPM packages..."
cd ../rpms
sudo dnf localinstall -y *.rpm
kubelet --version
kubeadm version
kubectl version --client
SCRIPT

chmod +x scripts/*.sh

# Step 6: Create bundle
echo "Step 6: Creating final bundle..."
cd ..
tar -czf k8s-offline-bundle-${K8S_VERSION}.tar.gz k8s-offline/

echo ""
echo "=========================================="
echo "Download Complete!"
echo "=========================================="
echo "Bundle created: k8s-offline-bundle-${K8S_VERSION}.tar.gz"
echo "Size: $(du -h k8s-offline-bundle-${K8S_VERSION}.tar.gz | cut -f1)"
echo ""
echo "Transfer this file to your offline environment and extract it."
echo "Then follow the scripts in the 'scripts' directory."
echo "=========================================="
```

Make it executable:

```bash
chmod +x complete-download.sh
./complete-download.sh
```

### Installation Script for Offline Environment

```bash
#!/bin/bash
# Script: offline-install.sh
# Purpose: Install Kubernetes in offline environment

set -e

echo "=========================================="
echo "Kubernetes Offline Installation"
echo "=========================================="

# Check if bundle exists
if [ ! -f "images/k8s-images-all.tar" ]; then
    echo "Error: Image archive not found!"
    echo "Please extract the offline bundle first."
    exit 1
fi

# Step 1: Load container images
echo "Step 1: Loading container images..."
if command -v docker &> /dev/null; then
    docker load -i images/k8s-images-all.tar
    docker images
elif command -v ctr &> /dev/null; then
    sudo ctr -n k8s.io image import images/k8s-images-all.tar
    sudo crictl images
else
    echo "Error: Neither docker nor containerd found!"
    exit 1
fi

# Step 2: Install RPM packages
echo "Step 2: Installing RPM packages..."
cd rpms
sudo dnf localinstall -y *.rpm
cd ..

# Step 3: Enable services
echo "Step 3: Enabling services..."
sudo systemctl enable --now containerd
sudo systemctl enable --now kubelet

# Verify installation
echo ""
echo "=========================================="
echo "Installation Complete!"
echo "=========================================="
kubelet --version
kubeadm version
kubectl version --client
echo ""
echo "Next steps:"
echo "1. Configure containerd"
echo "2. Initialize control plane or join worker node"
echo "3. Apply network plugin manifest from 'manifests' directory"
echo "=========================================="
```

---

## Quick Reference

### Download Everything (One Command)

```bash
# Run the complete download script
chmod +x complete-download.sh
./complete-download.sh

# Transfer to offline environment
scp k8s-offline-bundle-*.tar.gz user@offline-server:/tmp/
```

### Install in Offline Environment (One Command)

```bash
# Extract bundle
tar -xzf k8s-offline-bundle-*.tar.gz
cd k8s-offline

# Run installation
chmod +x scripts/*.sh
./scripts/02-load-images-containerd.sh  # or 01 for docker
./scripts/03-install-rpms.sh
```

---

## Verification

### Check Downloaded Images

```bash
# Count images in archive
tar -tzf images/k8s-images-all.tar | grep -c 'manifest.json'

# List images after loading
docker images | grep -E 'registry.k8s.io|calico|flannel'
# OR
sudo crictl images | grep -E 'registry.k8s.io|calico|flannel'
```

### Check Downloaded RPMs

```bash
# List RPMs
ls -lh rpms/*.rpm

# Count packages
ls rpms/*.rpm | wc -l

# Check if specific packages exist
ls rpms/ | grep -E 'kubelet|kubeadm|kubectl|containerd'
```

---

## Troubleshooting

### Issue: Images fail to pull

**Solution:**
```bash
# Try with different registry mirrors
docker pull registry.k8s.io/kube-apiserver:v1.31.0
# If fails, try:
docker pull k8s.gcr.io/kube-apiserver:v1.31.0
```

### Issue: RPM dependencies missing

**Solution:**
```bash
# Download with more dependencies
sudo dnf download --resolve --alldeps --downloadonly \
    --downloaddir=./rpms package-name
```

### Issue: Image load fails on offline system

**Solution:**
```bash
# Verify tar file integrity
tar -tzf k8s-images-all.tar | head

# Try loading individual images
for tar in images/*.tar; do
    echo "Loading $tar"
    docker load -i "$tar" || echo "Failed: $tar"
done
```

---

## Storage Requirements

Approximate sizes for complete offline bundle:

- **Container Images**: 2-4 GB
- **RPM Packages**: 500 MB - 1 GB
- **Manifests**: < 1 MB
- **Total Bundle**: ~3-5 GB compressed

Ensure you have at least **10 GB free space** on the download machine and **15 GB** on offline nodes.

---

## Summary

You now have everything needed to install Kubernetes completely offline:

✅ All container images downloaded and archived  
✅ All RPM packages with dependencies  
✅ Network plugin manifests  
✅ Installation scripts ready to use  

Transfer the bundle to your offline environment and run the installation scripts!

---

**Document Version**: 1.0  
**Last Updated**: November 2025  
**Compatible with**: RHEL 9.x, RHEL 10.x, Kubernetes 1.28+
