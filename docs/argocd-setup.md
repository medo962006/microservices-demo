```markdown
# ArgoCD Setup

## 1. Install ArgoCD
```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

## 2. Wait for all pods to be ready
```bash
kubectl get pods -n argocd -w
```

## 3. Access the ArgoCD API server
```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```
Keep this terminal open. Access the UI at https://localhost:8080 (accept the self‑signed certificate).

## 4. Retrieve the initial admin password
```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```
Copy the password.

## 5. Log in to the UI
- Username: `admin`
- Password: (the copied string)

## 6. Install ArgoCD CLI (optional but recommended)
- **macOS**: `brew install argocd`
- **Linux**: follow the [official instructions](https://argo-cd.readthedocs.io/en/stable/cli_installation/)
- **Windows**: download the binary from the [releases page](https://github.com/argoproj/argo-cd/releases)

Log in via CLI:
```bash
argocd login localhost:8080 --insecure --username admin
```
Enter the password when prompted.

## 7. Create an Application

### Using the UI
- Click **+ New App**
- **Application Name**: `online-boutique`
- **Project**: `default`
- **Sync Policy**: Manual (or enable Auto‑Sync if desired)
- **Repository URL**: `https://github.com/YOUR_USERNAME/microservices-demo.git`
- **Revision**: `HEAD` (or `main`)
- **Path**: `k8s-manifests`
- **Cluster URL**: `https://kubernetes.default.svc` (this refers to the cluster ArgoCD is installed in)
- **Namespace**: `default`
- Click **Create**, then **Sync**

### Using the CLI
```bash
argocd app create online-boutique \
  --repo https://github.com/YOUR_USERNAME/microservices-demo.git \
  --path k8s-manifests \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace default \
  --sync-policy automated
```

## 8. Verify the application
```bash
argocd app get online-boutique
```
Or check the UI: the application should become `Healthy` and `Synced`.

## 9. (Optional) Enable automatic sync
If you didn't enable auto‑sync during creation, you can set it later:
```bash
argocd app set online-boutique --sync-policy automated
```
```