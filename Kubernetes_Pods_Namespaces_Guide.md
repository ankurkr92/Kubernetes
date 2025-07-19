# 🧱 Kubernetes Pods & Namespaces

---

## 🔹 What is a Pod?
A **Pod** is the smallest deployable unit in Kubernetes.

It can contain one or more containers that:
- Share **network** (same IP address and port space)
- Share **storage** (volumes)
- Are designed to run closely related processes (e.g., helper/sidecar containers)

---

## 🔹 Pod Characteristics
- Each Pod gets **one IP address**—shared by all containers in that Pod.
- Containers within a Pod:
  - Can communicate via **localhost**
  - Share **volumes** and **memory namespaces**

---

## 🔹 Editing Existing Pods — Limitations
You **cannot fully edit** a running Pod’s specification.

Only these fields can be updated:
- `spec.containers[*].image`
- `spec.initContainers[*].image`
- `spec.activeDeadlineSeconds`
- `spec.tolerations`

> Other changes require deleting and re-creating the Pod.

---

## 🔹 Pod Creation Approaches

### 1️⃣ Imperative (Command Line)
```bash
kubectl run pod1 --image=nginx
```

### 2️⃣ Declarative (YAML Manifest)
**pod1.yaml**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod1
spec:
  containers:
  - name: nginx
    image: nginx
```

Apply the manifest:
```bash
kubectl apply -f pod1.yaml
```

---

## 🔹 Useful Pod Commands

| Task                            | Command                                  |
|---------------------------------|------------------------------------------|
| View Pods                       | `kubectl get pods`                       |
| View details                    | `kubectl describe pod pod1`             |
| Wide output (with IPs, nodes)   | `kubectl get pods -o wide`              |
| Access the container            | `kubectl exec -it pod1 -- /bin/bash`    |
| View Pod’s YAML                 | `kubectl get pod pod1 -o yaml`          |
| Delete Pod                      | `kubectl delete pod pod1`               |

---

## 🔹 Example: Modify NGINX Home Page
```bash
kubectl exec -it pod1 -- /bin/bash
cd /usr/share/nginx/html
echo "Hello nginx" > index.html
exit
```

Then check:
```bash
curl http://<Pod-IP>
# Output: Hello nginx
```

---

## 🔹 Generate Pod YAML with Dry Run
```bash
kubectl run pod1 --image=nginx --dry-run=client -o yaml > pod1.yaml
```

---

## 🔹 Update Pod Image
```bash
kubectl set image pod/pod1 nginx=nginx:1.25
```
or
```bash
kubectl edit pod pod1
# Modify the image field
```

---

## 🔹 Kubernetes Namespaces

Namespaces are Kubernetes virtual clusters within a physical cluster.

### ✅ Why Use Namespaces?
- To separate environments (e.g., dev, test, prod)
- To manage resources and policies independently
- To isolate teams and workloads

### 🛠 Create a Namespace
```bash
kubectl create namespace dev
```

### 📦 Run a Pod in a Namespace
```bash
kubectl run pod1 --image=nginx -n dev
```

### 🔍 View Resources in a Namespace
```bash
kubectl get pods -n dev
```

### 🧹 Delete a Namespace
```bash
kubectl delete namespace dev
```

---