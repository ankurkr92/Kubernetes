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

### On All Nodes - disable swap:

```bash
sudo apt update && sudo apt upgrade -y
sudo swapoff -a
sudo sed -i '/ swap / s/^/#/' /etc/fstab
```

### Load required modules:

```bash
cat << EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF

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
Without bridge-nf-call-iptables = 1:

[ Pod ] — bridge —> [ eth0 ]  
          ↪️ traffic bypasses iptables
With bridge-nf-call-iptables = 1:

[ Pod ] — bridge —> iptables —> [ eth0 ]  
          ✅ traffic is filtered by firewall rules
🧪 Real-world use in Kubernetes:
CNI plugins like Calico, Flannel, or Weave rely on iptables rules.

Without this setting, their traffic control and isolation fails silently.

Services, DNS, ingress controllers, etc., may not work correctly.

Ensures bridged container traffic passes through iptables so firewall rules and network policies work.

---

## 🐳 Step 2: Install Container Runtime (containerd)

```bash
sudo apt install -y containerd
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml
sudo systemctl restart containerd
sudo systemctl enable containerd
```

---

## 📦 Step 3: Install kubeadm, kubelet, kubectl

```bash
sudo apt-get update && sudo apt-get install -y apt-transport-https curl
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.32/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list > /dev/null
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.32/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

Note : if the command says gpg is not found, then install the gpg package ⇒ **apt install gpg** ]

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

Get the join command (this command is also printed during kubeadm init . Feel free to simply copy it from there).
```bash
kubeadm token create --print-join-command
```
Copy the join command from the control plane node. Run it on each worker node as root (i.e. with sudo ).

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
sudo systemctl stop kubelet
sudo systemctl stop containerd
sudo rm -rf /etc/kubernetes /var/lib/kubelet /var/lib/etcd /etc/cni/net.d ~/.kube
sudo iptables -F && sudo iptables -t nat -F && sudo iptables -t mangle -F && sudo iptables -X
sudo ipvsadm --clear  # Only if IPVS is installed
sudo systemctl start containerd
sudo systemctl start kubelet

CNI configuration is not removed. You must remove /etc/cni/net.d manually
sudo rm -rf /etc/cni/net.d

sudo iptables -F && sudo iptables -t nat -F && sudo iptables -t mangle -F && sudo iptables -X (iptables rules and IPVS tables are not cleaned,If you wish to reset iptables, you must do so manually)
```

---

## 📁 License

MIT

## 🙋‍♂️ Author

[Ankur Kumar Singh] - [ankurkr92@outlook.com]

---

Feel free to fork and contribute!

