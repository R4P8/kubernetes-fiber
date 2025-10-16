# 🚀 Go Fiber + PostgreSQL on Kubernetes

This project demonstrates a complete deployment of a **Go Fiber REST API** connected to a **PostgreSQL** database on **Kubernetes**.  
The setup uses `ConfigMap`, `Secret`, `Deployment`, `StatefulSet`, and `Service`, all managed via **Kustomize**.

---

## 🧱 Project Structure
```` 
kubernetes/
├── go-fiber/
│ ├── configmap.yaml
│ ├── deployment.yaml
│ ├── kustomization.yaml
│ ├── secret.yaml
│ └── service.yaml
└── postgre/
├── configmap.yaml
├── kustomization.yaml
├── secret.yaml
├── service.yaml
└── statefulset.yaml
```` 

## 🧩 Architecture Overview
````
+-------------------+ +-------------------+
| Go Fiber App | <----> | PostgreSQL DB |
| (go-fiber-deploy) | | (statefulset) |
| Port: 3000 | | Port: 5432 |
+-------------------+ +-------------------+
↑ ↑
ConfigMap & Secret ConfigMap & Secret
↓ ↓
Environment Vars Environment Vars
````

- The **Go Fiber** service (`go-fiber-http`) communicates internally with the **PostgreSQL** service (`postgres`) inside the `go-fiber` namespace.
- Configuration and credentials are injected via **ConfigMaps** and **Secrets**.

---

## ⚙️ Prerequisites

Before deploying, ensure you have:
- [Docker](https://www.docker.com/)
- [Minikube](https://minikube.sigs.k8s.io/)
- [kubectl](https://kubernetes.io/docs/tasks/tools/)
- [Kustomize](https://kubectl.docs.kubernetes.io/installation/kustomize/)

---

## 🗂️ Kubernetes Resources

### 🐘 PostgreSQL (Database)
Located in: `kubernetes/postgre/`

- **ConfigMap** → database host & port  
- **Secret** → database username, password, and DB name  
- **StatefulSet** → PostgreSQL pod with persistent volume  
- **Service** → internal ClusterIP service  
- **Kustomization** → combines all manifests for easy apply/delete  

Example deployment command:
````
kubectl apply -k kubernetes/postgre/
```` 
## ⚡ Go Fiber (Application)

Located in: ```` kubernetes/go-fiber/````

**Deployment** → runs Go Fiber REST API pods (replicas: 3)

**ConfigMap** → shared app configuration

**Secret** → reused PostgreSQL credentials

**Service** → ClusterIP exposing API internally (port 3000)

**Kustomization** → combines all manifests

Example deployment command:
`````
kubectl apply -k kubernetes/go-fiber/
```````
## 🚀 Deployment Guide
