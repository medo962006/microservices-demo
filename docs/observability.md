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