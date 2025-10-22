# 🚀 Go Fiber + PostgreSQL + + Observability Stack on Kubernetes

This project demonstrates a complete deployment of a **Go Fiber REST API** connected to a **PostgreSQL** and equipped with an observability stack **(OpenTelemetry Collector, Prometheus, Grafana, and Jaeger)**  on **Kubernetes**.  
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
├── otel-collector/
│   ├── configmap.yaml
│   ├── deployment.yaml
│   ├── service.yaml
│   └── kustomization.yaml
├── prometheus/
│   ├── configmap.yaml
│   ├── deployment.yaml
│   ├── service.yaml
│   └── kustomization.yaml
└── jaeger/
    ├── deployment.yaml
    ├── service.yaml
    └── kustomization.yaml
```` 

## 🧩 Architecture Overview
````
                               │
                ┌──────────────┴──────────────┐
                │        Prometheus           │
                │  Scrape app & node metrics  │
                └──────────────┬──────────────┘
                               │
                ┌──────────────┴──────────────┐
                │   OpenTelemetry Collector   │
                │ Collects + forwards traces  │
                │   (to Jaeger) & metrics     │
                └──────────────┬──────────────┘
                               │
                ┌──────────────┴──────────────┐
                │           Jaeger            │
                │  Trace backend (OTLP:4317)  │
                └──────────────┬──────────────┘
                               │
                ┌──────────────┴──────────────┐
                │         Go Fiber App        │
                │ Exposes metrics & tracing   │
                └──────────────┬──────────────┘
                               │
                ┌──────────────┴──────────────┐
                │        PostgreSQL DB        │
                └─────────────────────────────┘

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

**1. Start Minikube** 
````
kubectl apply -k kubernetes/go-fiber/
````
**2. Create Namespace** 
````
kubectl create namespace go-fiber
````
**3. Deploy PostgreSQL** 
````
kubectl apply -k kubernetes/postgre/
````
**4. Wait for PostgreSQL Pod to Run**
````
kubectl get pods -n go-fiber
````
**5. Deploy Go Fiber App**
````
kubectl apply -k kubernetes/go-fiber/
````
**6. Deploy OpenTelemetry Collector**
````
kubectl apply -k kubernetes/otel-collector/
````
**7. Deploy Jaeger**
````
kubectl apply -k kubernetes/jaeger/
````

**8. Verify All Resources**
````
kubectl get all -n go-fiber
````

## 📊 Observability Configuration
#### 🟣 OpenTelemetry Collector
* Receive traces from applications ````(otlp:4317)````

* Sending trace to Jaeger ````(jaeger:4317)````

* Exporting metrics to Prometheus ````(:9464)````

#### 🔵 Prometheus
````ConfigMap```` Example (````prometheus-config````):`
````
global:
  scrape_interval: 5s

scrape_configs:
  - job_name: 'otel-collector'
    static_configs:
      - targets: ['otel-collector:9464']

  - job_name: 'node-exporter'
    static_configs:
      - targets: ['node-exporter:9100']

  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
````
Prometheus collects metrics from:
* OpenTelemetry Collector (application Go Fiber)
* Node Exporter (resource node cluster)
* Herself

#### 🟠 Jaeger
* Jaeger UI is available on port 16686
* Receive trace via OTLP port 4317 from OpenTelemetry Collector

## 🌐 Port Summary

| Component               | Port(s)          | Description                  |
| ----------------------- | ---------------- | ---------------------------- |
| Go Fiber App            | 3000             | REST API                     |
| PostgreSQL              | 5432             | Database                     |
| OpenTelemetry Collector | 4317, 4318, 9464 | OTLP (GRPC/HTTP), Prometheus |
| Prometheus              | 9090             | Metrics visualization                       |
| Jaeger                  | 16686, 4317      | Web UI, OTLP traces          |


## 🔍 Testing the Application
**1. Port Forward the Go Fiber Service**
````
kubectl port-forward svc/go-fiber-http 3000:3000 -n go-fiber
````
**2. Access API Endpoint**
Visit:
````
http://127.0.0.1:3000/api/categories
````
**3. Expected Output**
````
{
  "data": [],
  "message": "Category list retrieved successfully"
}
````

## 🧠 Environment Variables Overview
| Variable      | Description             | Source    |
| ------------- | ----------------------- | --------- |
| `DB_HOST`     | PostgreSQL service name | ConfigMap |
| `DB_PORT`     | PostgreSQL port (5432)  | ConfigMap |
| `DB_USER`     | Database username       | Secret    |
| `DB_PASSWORD` | Database password       | Secret    |
| `DB_NAME`     | Database name           | Secret    |

## 🧩 Example Docker Image
Your Go Fiber app uses the following image (built from your app source):
````
image: repodocker/go-fiber-example:latest
````
You can build and push updates using:
````
docker build -t repodocker/go-fiber-example:latest .
docker push repodocker/go-fiber-example:latest
````
## 🧹 Cleanup
To delete all deployed resources:
````
kubectl delete -k kubernetes/go-fiber/
kubectl delete -k kubernetes/postgre/
````
Or delete the namespace entirely:
````
kubectl delete namespace go-fiber
````
## 🧾 Notes
• Use StatefulSet for PostgreSQL to ensure persistent data storage.

• Update Secret values with your own credentials for production use.

• The setup uses ClusterIP networking; for external access, use an Ingress or LoadBalancer type.

• Ensure both app and database share the same namespace: go-fiber.


