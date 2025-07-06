# How to Set Up a Kubernetes Cluster Using Kubeadm (Step-by-Step Guide)

This document provides a step-by-step guide to setting up a Kubernetes cluster using `kubeadm`.

---

## 🧰 Prerequisites

- Two or more Linux servers (Ubuntu 20.04+ recommended)
- Root or sudo privileges
- Access to the internet
- Swap disabled on all nodes

---

## 🔧 Step 1: Prepare the System

### On All Nodes:

```bash
sudo apt update && sudo apt upgrade -y
sudo swapoff -a
sudo sed -i '/ swap / s/^/#/' /etc/fstab
```

### Load required modules:

```bash
sudo modprobe overlay
sudo modprobe br_netfilter
```

### Add sysctl settings:

```bash
cat <<EOF | sudo tee /etc/sysctl.d/kubernetes.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF

sudo sysctl --system
```

---

## 🐳 Step 2: Install Container Runtime (containerd)

```bash
sudo apt install -y containerd
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml
sudo systemctl restart containerd
sudo systemctl enable containerd
```

---

## 📦 Step 3: Install kubeadm, kubelet, kubectl

```bash
sudo apt-get install -y apt-transport-https curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF

sudo apt update
sudo apt install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

---

## 🧱 Step 4: Initialize the Control Plane (on Master Node)

```bash
sudo kubeadm init --pod-network-cidr=192.168.0.0/16
```

After successful initialization, run:

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

---

## 🌐 Step 5: Install a Pod Network (Calico)

```bash
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/calico.yaml
```

---

## 🖥️ Step 6: Join Worker Nodes

On each worker node, run the join command shown at the end of the kubeadm init output:

```bash
sudo kubeadm join <master-ip>:6443 --token <token> \
    --discovery-token-ca-cert-hash sha256:<hash>
```

---

## ✅ Step 7: Verify the Cluster

On the master node:

```bash
kubectl get nodes
kubectl get pods -A
```

All nodes should be in `Ready` state.

---

## 🧹 (Optional) Reset Cluster

```bash
sudo kubeadm reset -f
sudo rm -rf /etc/cni/net.d /etc/kubernetes /var/lib/etcd ~/.kube
sudo iptables -F && sudo iptables -t nat -F && sudo iptables -t mangle -F && sudo iptables -X
```

---

## 📁 License

MIT

## 🙋‍♂️ Author

[Your Name] - [your-email\@example.com]

---

Feel free to fork and contribute!

