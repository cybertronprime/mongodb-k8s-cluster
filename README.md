# ☸️ MongoDB + Mongo Express on Kubernetes

> A fully containerized **MongoDB** database with **Mongo Express** admin UI, deployed and orchestrated on a local **Minikube** Kubernetes cluster.

---

## 🧩 How It All Connects — The Big Picture

```
                        ┌──────────────────────────────────────┐
                        │          MINIKUBE CLUSTER             │
                        │                                      │
  YOU (Browser)         │  ┌──────────────────────────────┐    │
  http://localhost:8081 │  │      Mongo Express Pod        │    │
        │               │  │   ┌──────────────────────┐    │    │
        │               │  │   │  mongo-express:latest │    │    │
        ▼               │  │   │  Port: 8081           │    │    │
┌──────────────────┐    │  │   └──────────────────────┘    │    │
│ mongo-express-   │◄───┼──┤                               │    │
│ service          │    │  └──────────────────────────────┘    │
│ (LoadBalancer)   │    │         │          │        │         │
│ Port: 8081       │    │         │ reads    │ reads  │ reads   │
└──────────────────┘    │         ▼          ▼        ▼         │
                        │  ┌─────────┐ ┌─────────┐ ┌─────────┐│
                        │  │ Secret  │ │ Secret  │ │ConfigMap ││
                        │  │(user)   │ │(pass)   │ │(db url)  ││
                        │  └─────────┘ └─────────┘ └────┬─────┘│
                        │                               │       │
                        │              resolves to ──────┘       │
                        │              "mongodb-service"         │
                        │                    │                   │
                        │                    ▼                   │
                        │  ┌──────────────────────────────┐     │
                        │  │     mongodb-service           │     │
                        │  │     (ClusterIP)               │     │
                        │  │     Port: 27017               │     │
                        │  └──────────────┬───────────────┘     │
                        │                 │                      │
                        │                 ▼                      │
                        │  ┌──────────────────────────────┐     │
                        │  │       MongoDB Pod             │     │
                        │  │  ┌──────────────────────┐     │     │
                        │  │  │   mongo:latest        │     │     │
                        │  │  │   Port: 27017         │     │     │
                        │  │  └──────────────────────┘     │     │
                        │  └──────────────────────────────┘     │
                        └──────────────────────────────────────┘
```

---

## 🔄 Request Flow — Step by Step

```
1. User opens http://localhost:8081
          │
          ▼
2. mongo-express-service (LoadBalancer) receives the request
          │
          ▼
3. Routes to Mongo Express Pod (port 8081)
          │
          ▼
4. Mongo Express reads credentials from Secret
   ├── ME_CONFIG_MONGODB_ADMINUSERNAME ← secret/mongodb-secret
   └── ME_CONFIG_MONGODB_ADMINPASSWORD ← secret/mongodb-secret
          │
          ▼
5. Mongo Express reads DB host from ConfigMap
   └── ME_CONFIG_MONGODB_SERVER ← configmap/mongodb-config → "mongodb-service"
          │
          ▼
6. Kubernetes DNS resolves "mongodb-service" → ClusterIP (10.x.x.x)
          │
          ▼
7. mongodb-service (ClusterIP:27017) routes to MongoDB Pod
          │
          ▼
8. MongoDB authenticates with the same Secret credentials
   ├── MONGO_INITDB_ROOT_USERNAME ← secret/mongodb-secret
   └── MONGO_INITDB_ROOT_PASSWORD ← secret/mongodb-secret
          │
          ▼
9. ✅ Connection established — data flows back to browser
```

---

## 🏗️ Deployment vs Service — How They Work Together

```
┌─────────────────────────────────────────────────────────────┐
│                        DEPLOYMENT                           │
│                                                             │
│  "I make sure the right number of Pods are always running"  │
│                                                             │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐                    │
│  │  Pod 1  │  │  Pod 2  │  │  Pod 3  │  ← replicas: 3     │
│  │ (app)   │  │ (app)   │  │ (app)   │                     │
│  └─────────┘  └─────────┘  └─────────┘                     │
│       ▲            ▲            ▲                           │
└───────┼────────────┼────────────┼───────────────────────────┘
        │            │            │
        └────────────┼────────────┘
                     │
              ┌──────┴──────┐
              │   SERVICE    │
              │              │
              │ "I provide   │
              │  a stable    │
              │  IP/DNS to   │
              │  reach the   │
              │  Pods"       │
              └──────────────┘
                     ▲
                     │
              Incoming Traffic

  🔹 ClusterIP  → Only reachable INSIDE the cluster (e.g. mongodb-service)
  🔹 LoadBalancer → Exposed OUTSIDE the cluster (e.g. mongo-express-service)
```

---

## 🔐 How Secrets & ConfigMaps Inject Into Pods

```
┌──────────────────────┐     ┌──────────────────────┐
│    Secret             │     │    ConfigMap          │
│  mongodb-secret       │     │  mongodb-config       │
│                       │     │                       │
│  username: YWRtaW4=   │     │  database_url:        │
│  password: ****       │     │    "mongodb-service"  │
└──────────┬───────────┘     └──────────┬───────────┘
           │                            │
           │   env[].valueFrom          │  env[].valueFrom
           │     .secretKeyRef          │    .configMapKeyRef
           ▼                            ▼
    ┌──────────────────────────────────────┐
    │              Pod Container            │
    │                                      │
    │  $USERNAME = "admin"   (decoded)     │
    │  $PASSWORD = "****"    (decoded)     │
    │  $DB_SERVER = "mongodb-service"      │
    └──────────────────────────────────────┘
```

---

## 📁 Project Structure

| File | K8s Resource | What It Does |
|------|-------------|--------------|
| `secret.yaml` | **Secret** | Stores base64-encoded MongoDB root username & password |
| `congifmap.yaml` | **ConfigMap** | Stores the internal MongoDB service URL (`mongodb-service`) |
| `deployment.yaml` | **Deployment + Service** | Runs MongoDB pod & exposes it internally via ClusterIP on port 27017 |
| `mongoexpress-deployment.yaml` | **Deployment + Service** | Runs Mongo Express pod & exposes it externally via LoadBalancer on port 8081 |

---

## ⚙️ Prerequisites

| Tool | Purpose |
|------|---------|
| [Docker Desktop](https://www.docker.com/products/docker-desktop/) | Container runtime |
| [Minikube](https://minikube.sigs.k8s.io/docs/start/) | Local Kubernetes cluster |
| [kubectl](https://kubernetes.io/docs/tasks/tools/) | Kubernetes CLI |

---

## 🏁 Getting Started

### 1️⃣ Start Minikube
```bash
minikube start
```

### 2️⃣ Apply resources (order matters!)
```bash
# Secret must exist before Deployments reference it
kubectl apply -f secret.yaml

# ConfigMap must exist before Mongo Express references it
kubectl apply -f congifmap.yaml

# MongoDB Deployment + ClusterIP Service
kubectl apply -f deployment.yaml

# Mongo Express Deployment + LoadBalancer Service
kubectl apply -f mongoexpress-deployment.yaml
```

### 3️⃣ Verify everything is running
```bash
kubectl get all
```

Expected output:
```
NAME                                       READY   STATUS    RESTARTS   AGE
pod/mondgodb-deployment-xxx                1/1     Running   0          ...
pod/mongodb-express-xxx                    1/1     Running   0          ...

NAME                            TYPE           CLUSTER-IP     PORT(S)
service/mongodb-service         ClusterIP      10.x.x.x      27017/TCP
service/mongo-express-service   LoadBalancer   10.x.x.x      8081:xxxxx/TCP
```

---

## 🌐 Access Mongo Express UI

```bash
minikube service mongo-express-service
```

This opens **Mongo Express** in your browser automatically. Alternatively:

```bash
kubectl port-forward deployment/mongodb-express 8081:8081
```

Then visit → **http://localhost:8081**

---

## 🔑 Secrets

| Key | Decoded Value |
|-----|---------------|
| `MONGO_INITDB_ROOT_USERNAME` | `admin` |
| `MONGO_INITDB_ROOT_PASSWORD` | *(encoded in secret.yaml)* |

> ⚠️ **Never commit real credentials to a public repo.** Use [Sealed Secrets](https://github.com/bitnami-labs/sealed-secrets) or an external secrets manager in production.

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

## 📚 Kubernetes Concepts Used

| Concept | Where Used |
|---------|-----------|
| **Deployment** | Manages MongoDB & Mongo Express pods |
| **Pod** | Smallest deployable unit — runs a single container |
| **Service (ClusterIP)** | Internal networking for MongoDB |
| **Service (LoadBalancer)** | External access for Mongo Express |
| **Secret** | Securely stores DB credentials (base64) |
| **ConfigMap** | Stores non-sensitive config (DB URL) |
| **Env Injection** | `secretKeyRef` & `configMapKeyRef` pass config into containers |
| **DNS Resolution** | `mongodb-service` resolves to MongoDB's ClusterIP inside the cluster |

---

