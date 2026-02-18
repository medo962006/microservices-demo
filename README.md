# Cloud-Native DevOps Platform: GitOps, CI/CD & Observability

[![CI](https://github.com/YOUR_USERNAME/microservices-demo/actions/workflows/ci.yaml/badge.svg)](https://github.com/YOUR_USERNAME/microservices-demo/actions/workflows/ci.yaml)

## 📌 Overview
This project demonstrates a complete DevOps pipeline for a microservices application on Kubernetes. It showcases:
- **Containerization** with Docker (multi‑stage builds, security best practices)
- **CI/CD** via GitHub Actions (automated build, vulnerability scan, image push)
- **GitOps** using ArgoCD (cluster state synced from Git)
- **Observability** with Prometheus (metrics), Loki (logs), and Grafana (unified dashboards)

All components run locally on Minikube, proving that the same principles apply to any cloud environment.

## 🏗️ Architecture
![Architecture Diagram](docs/images/architecture.png)

## 🧰 Technologies Used
| Area                | Tools                                                                 |
|---------------------|-----------------------------------------------------------------------|
| Containerization    | Docker (multi‑stage, non‑root)                                        |
| CI/CD               | GitHub Actions, Trivy (security scan)                                 |
| GitOps              | ArgoCD                                                                |
| Kubernetes          | Minikube, kubectl, Kustomize (optional)                               |
| Monitoring          | Prometheus, Grafana                                                   |
| Logging             | Loki, Promtail (or Fluent Bit)                                        |
| Application         | [Google Online Boutique](https://github.com/GoogleCloudPlatform/microservices-demo) (11 microservices) |

## 🚀 Getting Started

### Prerequisites
- Docker
- Minikube (with 4 CPUs, 8GB RAM)
- kubectl
- Helm
- Git

### Installation
1. **Clone the repository**
   ```bash
   git clone https://github.com/YOUR_USERNAME/microservices-demo.git
   cd microservices-demo
   ```

2. **Start Minikube**
   ```bash
   minikube start --cpus=4 --memory=8192
   ```

3. **Deploy the application**
   ```bash
   kubectl apply -f https://raw.githubusercontent.com/GoogleCloudPlatform/microservices-demo/main/release/kubernetes-manifests.yaml
   ```

4. **Access the frontend**
   ```bash
   kubectl port-forward service/frontend 8081:80
   ```
   Open http://localhost:8081

5. **Set up ArgoCD** (see detailed instructions in [docs/argocd-setup.md](docs/argocd-setup.md))

6. **Install observability stack** (Prometheus + Grafana + Loki) – refer to [docs/observability.md](docs/observability.md)

> **Note:** Detailed step‑by‑step guides are in the `docs/` folder.

## 🔄 CI/CD Pipeline
On every push to `main`, GitHub Actions:
- Builds the frontend Docker image
- Scans it for vulnerabilities (Trivy)
- Pushes it to Docker Hub
- Updates the Kubernetes manifest with the new image tag
- ArgoCD automatically syncs the cluster to the new state

![GitHub Actions Workflow](docs/images/ci-pipeline.png)

## 📊 Observability
- **Metrics:** Prometheus scrapes container metrics from kubelet. Custom dashboards in Grafana show CPU, memory, and request rates.
- **Logs:** Loki aggregates logs from all pods; queryable in Grafana.
- **Unified view:** A dedicated dashboard correlates metrics and logs for the frontend service.

![Grafana Dashboard](docs/images/grafana-dashboard.png)

## 🧠 Key Takeaways
- GitOps ensures declarative, auditable, and automated deployments.
- Observability is essential for running production systems – metrics and logs together tell the full story.
- The same pipeline can be extended to any cloud (AWS, GCP, Azure) with minimal changes.

## 📄 License
This project is for demonstration purposes. The original microservices-demo is Apache 2.0 licensed.

## 🙌 Acknowledgements
- [Google Cloud Platform](https://github.com/GoogleCloudPlatform/microservices-demo) for the sample application.
- The open‑source communities behind Docker, Kubernetes, ArgoCD, Prometheus, Loki, and Grafana.
```

---

### **docs/argocd-setup.md**

```markdown
# ArgoCD Setup

1. **Install ArgoCD**
   ```bash
   kubectl create namespace argocd
   kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
   ```

2. **Wait for all pods to be ready**
   ```bash
   kubectl get pods -n argocd -w
   ```

3. **Access the ArgoCD API server**
   ```bash
   kubectl port-forward svc/argocd-server -n argocd 8080:443
   ```
   Keep this terminal open.

4. **Retrieve the initial admin password**
   ```bash
   kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
   ```
   Copy the password.

5. **Log in to the UI**
   Open https://localhost:8080 in your browser (accept the self‑signed certificate).  
   Username: `admin`  
   Password: (the copied string)

6. **Create an application** (via UI or CLI)

   **Using the CLI:**
   ```bash
   argocd login localhost:8080 --insecure --username admin
   argocd app create online-boutique \
     --repo https://github.com/YOUR_USERNAME/microservices-demo.git \
     --path k8s-manifests \
     --dest-server https://kubernetes.default.svc \
     --dest-namespace default
   argocd app sync online-boutique
   ```

   **Using the UI:**
   - Click **+ New App**
   - Application Name: `online-boutique`
   - Project: `default`
   - Sync Policy: Manual (or enable Auto‑Sync)
   - Repository URL: `https://github.com/YOUR_USERNAME/microservices-demo.git`
   - Path: `k8s-manifests`
   - Cluster URL: `https://kubernetes.default.svc`
   - Namespace: `default`
   - Click **Create**, then **Sync**

7. **Verify** the application becomes `Healthy` and `Synced`.
```

---

### **docs/observability.md**

```markdown
# Observability Stack (Prometheus + Grafana + Loki)

## Install Prometheus & Grafana
```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
kubectl create namespace monitoring
helm install monitoring prometheus-community/kube-prometheus-stack -n monitoring
```

Wait for all pods to be ready:
```bash
kubectl get pods -n monitoring -w
```

## Install Loki
```bash
helm repo add grafana https://grafana.github.io/helm-charts
helm install loki grafana/loki-stack -n monitoring \
  --set fluent-bit.enabled=true \
  --set promtail.enabled=false
```

## Access Grafana
```bash
kubectl port-forward -n monitoring svc/monitoring-grafana 3000:80
```
Open http://localhost:3000  
Default credentials: `admin` / `prom-operator`

## Add Loki as a data source via API
With the port‑forward still running, run:
```bash
curl -X POST -H "Content-Type: application/json" \
  -d '{
    "name":"Loki",
    "type":"loki",
    "url":"http://loki.monitoring.svc.cluster.local:3100",
    "access":"proxy"
  }' \
  http://admin:prom-operator@localhost:3000/api/datasources
```

## Import the frontend dashboard
Save the following dashboard JSON as `dashboard.json` in the `docs/` folder:

```json
{
  "dashboard": {
    "title": "Online Boutique Frontend",
    "tags": ["online-boutique", "kubernetes"],
    "timezone": "browser",
    "panels": [
      {
        "id": 1,
        "title": "Frontend CPU Usage (5m rate)",
        "type": "graph",
        "datasource": "Prometheus",
        "gridPos": {"h": 8, "w": 12, "x": 0, "y": 0},
        "targets": [
          {
            "expr": "rate(container_cpu_usage_seconds_total{namespace=\"default\", pod=~\"frontend-.*\"}[5m])",
            "legendFormat": "{{pod}} - CPU cores"
          }
        ]
      },
      {
        "id": 2,
        "title": "Frontend Memory Usage",
        "type": "graph",
        "datasource": "Prometheus",
        "gridPos": {"h": 8, "w": 12, "x": 12, "y": 0},
        "targets": [
          {
            "expr": "container_memory_working_set_bytes{namespace=\"default\", pod=~\"frontend-.*\"}",
            "legendFormat": "{{pod}} - memory bytes"
          }
        ]
      },
      {
        "id": 3,
        "title": "Frontend Logs",
        "type": "logs",
        "datasource": "Loki",
        "gridPos": {"h": 8, "w": 24, "x": 0, "y": 8},
        "targets": [
          {
            "expr": "{container=\"server\", namespace=\"default\"}",
            "refId": "A"
          }
        ]
      }
    ]
  },
  "overwrite": true
}
```

Then import it:
```bash
curl -X POST -H "Content-Type: application/json" \
  -d @dashboard.json \
  http://admin:prom-operator@localhost:3000/api/dashboards/db
```

## Verify
- Generate traffic to the frontend (visit the store, click around)
- Open Grafana, find the "Online Boutique Frontend" dashboard
- You should see CPU and memory graphs, and logs from the frontend container.
```

---