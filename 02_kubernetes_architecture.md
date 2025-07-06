# Kubernetes Cluster Architecture (Step-by-Step Guide)

This document explains the architecture of a Kubernetes cluster, component by component, to help beginners and practitioners understand how Kubernetes works under the hood.

---

## 🧱 What is a Kubernetes Cluster?

A Kubernetes cluster is a set of nodes that run containerized applications. It consists of at least one **control plane** and multiple **worker nodes**.

---

## 🌐 Components of a Kubernetes Cluster

### 1. **Control Plane Components** (Master Node)

These components manage the overall cluster state and orchestration.

#### 🧠 kube-apiserver

- Acts as the front-end for the Kubernetes control plane.
- All `kubectl` commands go through this API server.

#### 📋 etcd

- A distributed, consistent key-value store.
- Stores all Kubernetes cluster data (state).

#### 🧭 kube-scheduler

- Assigns newly created pods to suitable worker nodes based on resource requirements and constraints.

#### 🎮 kube-controller-manager

- Runs multiple controllers (e.g., Node Controller, Replication Controller).
- Ensures desired state matches the current state.

#### 🔐 cloud-controller-manager *(optional)*

- Integrates Kubernetes with cloud provider APIs (e.g., AWS, GCP, Azure).

---

### 2. **Node Components** (Worker Nodes)

These run the containerized applications.

#### 🐳 container runtime

- Software that runs containers (e.g., containerd, Docker).

#### 🧰 kubelet

- Agent running on each node.
- Communicates with the control plane.
- Ensures containers are running in a pod.

#### 🌉 kube-proxy

- Maintains network rules on nodes.
- Enables pod-to-pod and pod-to-service communication.

---

## 📦 Add-Ons (Optional but Common)

#### 🕸️ CoreDNS

- Provides DNS services for the cluster.

#### 📡 Network Plugin (CNI)

- Enables networking for pods (e.g., Calico, Flannel, Cilium).

#### 📊 Metrics Server

- Collects resource metrics (CPU, memory) from nodes and pods.

---

## 🖼️ Visual Overview

![image](https://github.com/user-attachments/assets/e12a8d1b-de12-40d4-b71b-9c3d4b7051fe)

![image](https://github.com/user-attachments/assets/5022696f-c51a-4138-ba63-94de2bf9c90b)

> Source: [Kubernetes Official Documentation](https://kubernetes.io/docs/concepts/architecture/)

---

## ✅ Summary

- The **control plane** manages the cluster.
- **Worker nodes** run containerized apps.
- All components work together to automate deployment, scaling, and operations.

---

## 📁 License

MIT

## 🙋‍♂️ Author

[Ankur Kumar Singh  ] - [[ankurkr92@outlook.com](mailto:your-email@example.com)]

---

Feel free to fork and contribute!

