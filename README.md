# ArgoCD Lab

## Architecture
```
┌─────────────────────────────────────────┐
│             argocd-lab                  │
├──────────┬──────────────────────────────┤
│          │                              │
│ /argocd  │     /app-of-apps             │
│          │      (Bootstrap)             │
│          │          │                   │
│          │      /gitops                 │
│          │          │                   │
│          │     ApplicationSet           │
│          │                              │
└────┬─────┴──────┬───────────────────────┘
     │            │
  ArgoCD         App-of-Apps
  Installation   (Gitops Layer)
     │            │
     └────┬───────┘
          │
      Kubernetes

```

### Layers

1. `/argocd`
   - Creates namespaces
   - Installs and configures ArgoCD

2. `/app-of-apps`
   - Contains the bootstrap Application that points to the gitops layer

3. `/app-of-apps/gitops`
   - Contains the ApplicationSet that dynamically discovers and manages applications

---

## Structure

```
argocd-lab/
├── README.md                              # This file
│
├── argocd/                                # ArgoCD Installation & Configuration
│   ├── kustomization.yaml                 # Base Kustomize configuration
│   └── base/
│       └── namespace.yaml                 # argocd namespace definition
│
└── app-of-apps/                           # App-of-Apps Bootstrap & GitOps Layer
    ├── gitops-app.yaml                    # Bootstrap Application (entry point)
    └── gitops/                            # GitOps configuration layer
        ├── kustomization.yaml             # Kustomize base
        └── applicationsets/
            └── apps-git-generator.yaml    # ApplicationSet with git generator
```

## Getting Started

### Prerequisites

- minikube cluster (v1.38+)
- `kubectl` configured to access your cluster
- `git` installed and configured

### Installation

#### 1. Clone

```bash
git clone https://github.com/ericmolinam/argocd-lab.git
cd argocd-lab
```

#### 2. Deploy ArgoCD

Deploy the ArgoCD core installation:

```bash
kubectl apply --server-side -k argocd/
```

This command:
- Creates the `argocd` namespace
- Installs ArgoCD using the official manifests (v3.3.1)

Wait for ArgoCD to be ready before proceeding:

```bash
kubectl wait --for=condition=available --timeout=300s deployment/argocd-server -n argocd
```

#### 3. Deploy the App-of-Apps Bootstrap

```bash
kubectl apply -f app-of-apps/gitops-app.yaml -n argocd
```

#### 4. Access ArgoCD UI

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Open: `https://localhost:8080`

**Credentials:**
- Username: `admin`
- Password:
  ```bash
  kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 -d
  ```