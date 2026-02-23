# ArgoCD Lab

## Architecture
```
┌────────────────────────────────────────┐
│             argocd-lab                 │
├──────────┬──────────────┬──────────────┤
│          │              │              │
│ /argocd  │ /gitops      │ /apps        │
│          │              │              │
└────┬─────┴──────┬───────┴────────┬─────┘
     │            │                │
     │            │             ┌──┴──────────────┐
     │            │             │ - bookinfo/     │
     │            │             │ - podinfo/      │
     │            │             │ - [more apps]   │
     │            │             └─────────────────┘
     │            │
  ArgoCD         GitOps (app-of-apps)
  Installation  ApplicationSet
     │            │
     └────┬───────┘
          │
      Kubernetes

```

### Layers

1. `/argocd`
   - Installs and configures ArgoCD itself
   - Creates the initial "gitops" application that bootstraps everything else
   - Uses the app-of-apps pattern to manage itself and other apps

2. `/gitops`
   - Contains the ApplicationSet that dynamically discovers applications
   - Uses git generator to scan `/apps` directory for new applications
   - Automatically creates ArgoCD Applications for each discovered app

3. `/apps`
   - Contains individual applications (bookinfo, podinfo)
   - Each app has its own Kubernetes manifests

---

## Structure

```
argocd-lab/
├── README.md                              # This file
│
├── argocd/                                # ArgoCD Installation & Configuration
│   ├── kustomization.yaml                 # Base Kustomize configuration
│   ├── namespace.yaml                     # argocd namespace
│   └── app-of-apps/
│       ├── namespace.yaml                 # gitops namespace
│       └── argocd-app.yaml                # Bootstraps the gitops app
│
├── gitops/                                # App-of-Apps & ApplicationSets
│   ├── kustomization.yaml                 # Kustomize base
│   └── applicationsets/
│       └── apps-git.yaml                  # ApplicationSet with git generator
│
└── apps/                                  # Deployed Applications
    ├── bookinfo/                          # Istio BookInfo sample app
    │   ├── kustomization.yaml
    │   └── base/
    │       ├── productpage.yaml
    │       ├── details.yaml
    │       ├── ratings.yaml
    │       └── reviews.yaml
    │
    └── podinfo/                           # Cloud native demo app
        ├── kustomization.yaml
        └── base/
            ├── deployment.yaml
            ├── service.yaml
            └── hpa.yaml
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

Once ArgoCD is running, deploy the app-of-apps that bootstraps the gitops layer:

```bash
kubectl apply -f argocd/app-of-apps/argocd-app.yaml
```

This deploys the bootstrap Application that enables ArgoCD to manage itself and other applications.

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