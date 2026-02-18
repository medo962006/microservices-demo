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