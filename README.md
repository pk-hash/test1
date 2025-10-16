# Infrastructure Repository

GitOps repository for managing ArgoCD and infrastructure components.

## Structure

```
.
├── apps/                    # ArgoCD Application definitions (App of Apps)
│   ├── root.yaml           # Root application (manages all apps)
│   └── example-app.yaml    # Example application
├── bootstrap/              # ArgoCD installation and bootstrap
│   └── argocd/            # ArgoCD installation manifests
└── manifests/             # Kubernetes manifests for applications
```

## Quick Start

### 1. Install ArgoCD

```bash
# Create namespace
kubectl create namespace argocd

# Install ArgoCD (choose one method)
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Wait for pods to be ready
kubectl wait --for=condition=Ready pods --all -n argocd --timeout=300s
```

### 2. Access ArgoCD UI

```bash
# Port forward
kubectl port-forward svc/argocd-server -n argocd 8080:443

# Get initial admin password
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

### 3. Deploy Root Application

Update `apps/root.yaml` with your repository URL, then:

```bash
# Apply the root app (App of Apps pattern)
kubectl apply -f apps/root.yaml

# This will deploy all applications defined in the apps/ directory
```

## Creating New Applications

Create a new Application manifest in `apps/`:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/YOUR_ORG/YOUR_REPO.git
    targetRevision: HEAD
    path: manifests/my-app
  destination:
    server: https://kubernetes.default.svc
    namespace: my-app
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
```

## ArgoCD CLI

```bash
# Login
argocd login localhost:8080

# List apps
argocd app list

# Sync an app
argocd app sync my-app

# Get app details
argocd app get my-app
```

## Next Steps

1. Update repository URLs in all Application manifests
2. Configure ArgoCD RBAC and SSO (in `bootstrap/argocd/`)
3. Create AppProjects for multi-tenancy
4. Add your infrastructure manifests
5. Set up CI/CD pipelines for automated deployments
