# 🚀 Kubernetes Practice Setup (Minikube on EC2)

## 📌 Overview

This project documents setting up a Kubernetes environment using **Minikube on an AWS EC2 instance**, deploying applications, exposing services, and debugging networking issues.

---

## ⚙️ Environment Setup

* **Cloud**: AWS EC2 (Ubuntu)
* **Kubernetes**: Minikube (Docker driver)
* **Container Runtime**: Docker
* **CLI Tools**: kubectl, minikube

---

## 🛠️ Installation Steps

### 1. Install Docker

```bash
sudo apt update
sudo apt install -y docker.io
sudo systemctl enable docker
sudo systemctl start docker
sudo usermod -aG docker ubuntu
```

---

### 2. Install Minikube

```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

---

### 3. Install kubectl

```bash
sudo apt install -y apt-transport-https ca-certificates curl gnupg
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | \
gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] \
https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /" | \
sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt update
sudo apt install -y kubectl
```

---

### 4. Start Minikube

```bash
minikube start --driver=docker
```

---

## 🚀 Deployment

### Deployment YAML

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: python-web-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: python-web-app
  template:
    metadata:
      labels:
        app: python-web-app
    spec:
      containers:
      - name: python-web-app
        image: abhishekf5/python-sample-app-demo:v1
        ports:
        - containerPort: 8000
```

---

## 🌐 Service Exposure

### NodePort Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: NodePort
  selector:
    app: python-web-app
  ports:
    - port: 80
      targetPort: 8000
      nodePort: 30007
```

---

### LoadBalancer Service

```yaml
spec:
  type: LoadBalancer
```

⚠️ In Minikube:

* EXTERNAL-IP is internal (10.x.x.x)
* Not accessible from outside

---

## 🔴 Issues Faced

### 1. Image Pull Failure

**Error:**

```
no space left on device
```

**Fix:**

```bash
docker system prune -a -f
minikube delete
```

---

### 2. Service Not Accessible Externally

Tried:

```text
http://<EC2-IP>:30007
```

❌ Not working

---

## 🔍 Root Cause

Minikube (Docker driver) runs Kubernetes inside a **Docker network (NAT)**:

```text
EC2 Network ≠ Minikube Network
```

So:

* NodePort binds to internal IP (192.168.x.x)
* Not exposed to EC2 public interface

---

## ✅ Solution

### Use Port Forwarding

```bash
kubectl port-forward service/my-service 8000:80 --address 0.0.0.0
```

---

### Access Application

```text
http://<EC2-PUBLIC-IP>:8000
```

---

## 🔁 Networking Flow

```text
Laptop → EC2:8000 → Service:80 → Pod:8000
```

---

## 🧠 Key Learnings

* `containerPort` = internal app port (does not expose)
* `targetPort` = actual pod port
* `port` = service internal port
* `nodePort` = external access (only in real clusters)

---

## ⚠️ Important Concepts

### Why NodePort Failed

* Bound to Minikube internal network
* Not accessible externally

### Why LoadBalancer Failed

* No real cloud LB in Minikube
* Only internal IP assigned

---

## 🔥 Working Method

```bash
kubectl port-forward service/my-service 8000:80 --address 0.0.0.0
```

---

## 📌 Final Takeaway

👉 Minikube (Docker driver) isolates networking
👉 NodePort/LoadBalancer may not work as expected
👉 Port-forward is the most reliable method in this setup

---

## 🚀 Future Improvements

* Use `minikube --driver=none`
* Use `k3s` or `kubeadm`
* Move to AWS EKS for real-world setup

---

## 💡 One-Line Summary

**“Due to Minikube’s Docker-based network isolation, external access via NodePort failed. This was resolved using kubectl port-forward to expose the service on the EC2 interface.”**

---

