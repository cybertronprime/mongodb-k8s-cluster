# 🚀 MongoDB + Mongo Express on Kubernetes

A complete **Kubernetes** deployment for running **MongoDB** with a **Mongo Express** web-based admin UI, managed locally via **Minikube**.

---

## 📐 Architecture

```
┌─────────────────────────────────────────────────────┐
│                   Minikube Cluster                   │
│                                                     │
│  ┌─────────────┐         ┌──────────────────────┐  │
│  │   Secret     │────────▶│  MongoDB Deployment  │  │
│  │  (creds)     │────┐   │  (mongo:latest)      │  │
│  └─────────────┘    │   │  Port: 27017          │  │
│                      │   └──────────┬───────────┘  │
│  ┌─────────────┐    │              │               │
│  │  ConfigMap   │──┐ │    ┌────────▼────────┐      │
│  │ (db url)     │  │ │    │ MongoDB Service │      │
│  └─────────────┘  │ │    │  (ClusterIP)    │      │
│                    │ │    └────────┬────────┘      │
│                    │ │             │                │
│                    ▼ ▼             ▼                │
│            ┌───────────────────────────┐           │
│            │  Mongo Express Deployment │           │
│            │  (mongo-express:latest)   │           │
│            │  Port: 8081              │           │
│            └───────────────────────────┘           │
└─────────────────────────────────────────────────────┘
```

---

## 📁 Project Structure

| File | Kind | Description |
|------|------|-------------|
| `secret.yaml` | **Secret** | Base64-encoded MongoDB root credentials |
| `congifmap.yaml` | **ConfigMap** | Stores the MongoDB service URL |
| `deployment.yaml` | **Deployment + Service** | MongoDB database & internal ClusterIP service |
| `mongoexpress-deployment.yaml` | **Deployment** | Mongo Express web UI connected to MongoDB |

---

## ⚙️ Prerequisites

- [Docker Desktop](https://www.docker.com/products/docker-desktop/)
- [Minikube](https://minikube.sigs.k8s.io/docs/start/)
- [kubectl](https://kubernetes.io/docs/tasks/tools/)

---

## 🏁 Getting Started

### 1. Start Minikube
```bash
minikube start
```

### 2. Apply resources (order matters!)
```bash
# 1️⃣  Secret (credentials must exist first)
kubectl apply -f secret.yaml

# 2️⃣  ConfigMap (DB URL reference)
kubectl apply -f congifmap.yaml

# 3️⃣  MongoDB Deployment & Service
kubectl apply -f deployment.yaml

# 4️⃣  Mongo Express Deployment
kubectl apply -f mongoexpress-deployment.yaml
```

### 3. Verify everything is running
```bash
kubectl get all
```

---

## 🌐 Access Mongo Express UI

Once all pods are running, expose the Mongo Express service:

```bash
kubectl port-forward deployment/mongodb-express 8081:8081
```

Then open **http://localhost:8081** in your browser.

---

## 🔑 Secrets Reference

| Key | Decoded Value |
|-----|---------------|
| `MONGO_INITDB_ROOT_USERNAME` | `admin` |
| `MONGO_INITDB_ROOT_PASSWORD` | *(base64 encoded in secret.yaml)* |

> ⚠️ **Never commit real credentials to a public repo.** Use sealed secrets or an external secret manager in production.

---

## 🧹 Cleanup

```bash
kubectl delete -f mongoexpress-deployment.yaml
kubectl delete -f deployment.yaml
kubectl delete -f congifmap.yaml
kubectl delete -f secret.yaml
minikube stop
```

---

## 📚 Concepts Covered

- Kubernetes **Deployments** & **Pods**
- **Services** (ClusterIP)
- **Secrets** for sensitive data
- **ConfigMaps** for configuration
- **Environment variable injection** via `secretKeyRef` & `configMapKeyRef`
- Local development with **Minikube**

---

<p align="center">Made with ☸️ by <strong>Rohit Prasad</strong></p>
