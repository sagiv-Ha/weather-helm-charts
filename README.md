# 🌦️ Weather App GitOps Helm Repository

![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white)
![Helm](https://img.shields.io/badge/Helm-0F1689?style=for-the-badge&logo=helm&logoColor=white)
![ArgoCD](https://img.shields.io/badge/ArgoCD-EF7B4D?style=for-the-badge&logo=argo&logoColor=white)
![GitOps](https://img.shields.io/badge/GitOps-Automated-blueviolet?style=for-the-badge)
![Docker Hub](https://img.shields.io/badge/Docker%20Hub-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![GitHub Actions](https://img.shields.io/badge/GitHub%20Actions-2088FF?style=for-the-badge&logo=github-actions&logoColor=white)

---

## 📌 Overview

This repository contains the **Helm Charts** and **ArgoCD Application manifests** used to deploy the Weather App microservices into Kubernetes using a GitOps workflow.

The Weather App is composed of two microservices:

| Service | Description | Docker Image |
|---|---|---|
| `weather-backend` | Flask backend service that fetches weather data from OpenWeatherMap API | `harirs/weather-backend` |
| `weather-frontend` | Frontend service that displays weather information to users | `harirs/weather-frontend` |

This repository is the **GitOps source of truth** for Kubernetes deployments.

ArgoCD watches this repository and automatically synchronizes the Kubernetes cluster whenever a change is pushed to Git.

---

## 🧰 Technologies Used

| Tool | Purpose |
|---|---|
| **Kubernetes** | Runs the application workloads |
| **Helm** | Templates Kubernetes manifests |
| **ArgoCD** | Performs GitOps synchronization |
| **Docker Hub** | Stores container images |
| **GitHub Actions** | Builds images and updates Helm values |
| **OpenWeatherMap API** | Provides live weather data |

---

## 🧱 Repository Structure

```text
weather-helm-charts/
│
├── README.md
├── .gitignore
│
├── examples/
│   └── weather-backend-secret.example.env
│
├── charts/
│   ├── weather-backend/
│   │   ├── Chart.yaml
│   │   ├── values-dev.yaml
│   │   ├── values-prod.yaml
│   │   └── templates/
│   │       ├── deployment.yaml
│   │       └── service.yaml
│   │
│   └── weather-frontend/
│       ├── Chart.yaml
│       ├── values-dev.yaml
│       ├── values-prod.yaml
│       └── templates/
│           ├── deployment.yaml
│           └── service.yaml
│
└── argocd/
    ├── weather-backend-dev.yaml
    ├── weather-backend-prod.yaml
    ├── weather-frontend-dev.yaml
    └── weather-frontend-prod.yaml
```

---

## 🚀 GitOps Architecture

The deployment flow is:

```text
Developer pushes code
        ↓
GitHub Actions builds Docker image
        ↓
Image is pushed to Docker Hub
        ↓
GitHub Actions updates image tag / values in this Helm repository
        ↓
ArgoCD detects changes in this repository
        ↓
ArgoCD syncs the desired state to Kubernetes
        ↓
Application is deployed to dev/prod namespaces
```

---

## 🔁 Important GitOps Concept

ArgoCD does **not** deploy directly from the application source code repositories.

ArgoCD watches this repository:

```text
weather-helm-charts
```

This repository represents the desired Kubernetes state.

The application source code is stored separately in:

```text
weather-backend
weather-frontend
```

The Docker images are stored in Docker Hub:

```text
harirs/weather-backend
harirs/weather-frontend
```

The correct flow is:

```text
Application Repo → GitHub Actions → Docker Hub → Helm Repo → ArgoCD → Kubernetes
```

---

## 🌍 Environments

This project uses two Kubernetes environments:

| Environment | Namespace | Values File |
|---|---|---|
| Development | `dev` | `values-dev.yaml` |
| Production | `prod` | `values-prod.yaml` |

Each microservice has its own Helm chart and its own values file per environment.

---

# 📦 Helm Charts

The repository contains two separate Helm charts:

```text
charts/weather-backend
charts/weather-frontend
```

Each microservice has its own chart, which makes the deployment modular and easier to manage.

---

# 🔙 Backend Helm Chart

Path:

```text
charts/weather-backend
```

## Backend Chart Structure

```text
charts/weather-backend/
├── Chart.yaml
├── values-dev.yaml
├── values-prod.yaml
└── templates/
    ├── deployment.yaml
    └── service.yaml
```

---

## `charts/weather-backend/Chart.yaml`

This file defines the metadata of the backend Helm chart.

Example:

```yaml
apiVersion: v2
name: weather-backend
description: Helm chart for Weather Backend service
type: application
version: 0.1.0
appVersion: "1.0.0"
```

### Field Explanation

| Field | Description |
|---|---|
| `apiVersion` | Helm chart API version. `v2` is used for modern Helm charts. |
| `name` | Name of the Helm chart. |
| `description` | Short description of the chart. |
| `type` | Chart type. `application` means it deploys an application. |
| `version` | Version of the Helm chart itself. |
| `appVersion` | Version of the application being deployed. |

---

## `charts/weather-backend/values-dev.yaml`

This file contains backend configuration for the `dev` environment.

Example:

```yaml
namespace: dev

replicas: 1

image:
  repository: harirs/weather-backend
  tag: dev
  pullPolicy: Always

service:
  type: ClusterIP
  port: 5000

secret:
  name: weather-backend-secret
  key: OPENWEATHER_API_KEY
```

### Backend Dev Values Explanation

| Value | Description |
|---|---|
| `namespace` | Kubernetes namespace where the backend will be deployed. |
| `replicas` | Number of backend Pods. |
| `image.repository` | Docker Hub image repository. |
| `image.tag` | Image tag used for the dev environment. |
| `image.pullPolicy` | Defines when Kubernetes should pull the image. |
| `service.type` | Kubernetes Service type. |
| `service.port` | Port exposed by the backend service. |
| `secret.name` | Name of the Kubernetes Secret containing the OpenWeatherMap API key. |
| `secret.key` | Key inside the Kubernetes Secret. |

In development, the backend uses:

```text
harirs/weather-backend:dev
```

---

## `charts/weather-backend/values-prod.yaml`

This file contains backend configuration for the `prod` environment.

Example:

```yaml
namespace: prod

replicas: 1

image:
  repository: harirs/weather-backend
  tag: prod
  pullPolicy: Always

service:
  type: ClusterIP
  port: 5000

secret:
  name: weather-backend-secret
  key: OPENWEATHER_API_KEY
```

### Backend Prod Values Explanation

| Value | Description |
|---|---|
| `namespace` | Kubernetes namespace where the backend will be deployed. |
| `replicas` | Number of backend Pods in production. |
| `image.repository` | Docker Hub backend image repository. |
| `image.tag` | Production image tag. |
| `image.pullPolicy` | Defines image pull behavior. |
| `service.type` | Internal Kubernetes Service type. |
| `service.port` | Backend service port. |
| `secret.name` | Kubernetes Secret name. |
| `secret.key` | Secret key used by the backend container. |

In production, the backend uses:

```text
harirs/weather-backend:prod
```

---

## `charts/weather-backend/templates/deployment.yaml`

This template creates the Kubernetes Deployment for the backend.

It is responsible for:

- Creating the backend Pod.
- Pulling the backend Docker image.
- Setting the number of replicas.
- Exposing container port `5000`.
- Injecting the OpenWeatherMap API key from a Kubernetes Secret.

Important section:

```yaml
env:
  - name: OPENWEATHER_API_KEY
    valueFrom:
      secretKeyRef:
        name: weather-backend-secret
        key: OPENWEATHER_API_KEY
```

This means the API key is **not stored in Git**.

Instead, Kubernetes reads it from a Secret named:

```text
weather-backend-secret
```

---

## `charts/weather-backend/templates/service.yaml`

This template creates a Kubernetes Service for the backend.

The backend service allows other Pods in the same namespace to access the backend using:

```text
http://weather-backend:5000
```

This is used by the frontend service.

The service type is:

```text
ClusterIP
```

This means the backend is exposed only inside the Kubernetes cluster.

---

# 🖥️ Frontend Helm Chart

Path:

```text
charts/weather-frontend
```

## Frontend Chart Structure

```text
charts/weather-frontend/
├── Chart.yaml
├── values-dev.yaml
├── values-prod.yaml
└── templates/
    ├── deployment.yaml
    └── service.yaml
```

---

## `charts/weather-frontend/Chart.yaml`

This file defines metadata for the frontend Helm chart.

Example:

```yaml
apiVersion: v2
name: weather-frontend
description: Helm chart for Weather Frontend service
type: application
version: 0.1.0
appVersion: "1.0.0"
```

### Field Explanation

| Field | Description |
|---|---|
| `apiVersion` | Helm chart API version. |
| `name` | Name of the frontend chart. |
| `description` | Description of the chart. |
| `type` | Chart type. |
| `version` | Version of the Helm chart. |
| `appVersion` | Version of the frontend application. |

---

## `charts/weather-frontend/values-dev.yaml`

This file contains frontend configuration for the `dev` namespace.

Example:

```yaml
namespace: dev

replicas: 1

image:
  repository: harirs/weather-frontend
  tag: dev
  pullPolicy: IfNotPresent

service:
  type: ClusterIP
  port: 5001

env:
  BACKEND_URL: "http://weather-backend:5000"
```

### Frontend Dev Values Explanation

| Value | Description |
|---|---|
| `namespace` | Kubernetes namespace for the frontend. |
| `replicas` | Number of frontend Pods. |
| `image.repository` | Docker Hub frontend image repository. |
| `image.tag` | Image tag used for the dev environment. |
| `image.pullPolicy` | Defines image pull behavior. |
| `service.type` | Kubernetes Service type. |
| `service.port` | Frontend service port. |
| `env.BACKEND_URL` | Internal backend URL used by the frontend. |

The frontend connects to the backend using:

```text
http://weather-backend:5000
```

This works because both frontend and backend run in the same namespace.

---

## `charts/weather-frontend/values-prod.yaml`

This file contains frontend configuration for the `prod` namespace.

Example:

```yaml
namespace: prod

replicas: 1

image:
  repository: harirs/weather-frontend
  tag: prod
  pullPolicy: IfNotPresent

service:
  type: ClusterIP
  port: 5001

env:
  BACKEND_URL: "http://weather-backend:5000"
```

### Frontend Prod Values Explanation

| Value | Description |
|---|---|
| `namespace` | Kubernetes namespace for production. |
| `replicas` | Number of frontend Pods in production. |
| `image.repository` | Docker Hub frontend image repository. |
| `image.tag` | Production frontend image tag. |
| `image.pullPolicy` | Defines image pull behavior. |
| `service.type` | Kubernetes Service type. |
| `service.port` | Port exposed by the frontend service. |
| `env.BACKEND_URL` | Internal backend URL. |

In production, the frontend uses:

```text
harirs/weather-frontend:prod
```

---

## `charts/weather-frontend/templates/deployment.yaml`

This template creates the Kubernetes Deployment for the frontend.

It defines:

- Frontend Docker image.
- Number of replicas.
- Container port.
- Backend URL environment variable.

Important section:

```yaml
env:
  - name: BACKEND_URL
    value: "http://weather-backend:5000"
```

The frontend uses this environment variable to call the backend.

---

## `charts/weather-frontend/templates/service.yaml`

This template creates a Kubernetes Service for the frontend.

The service exposes the frontend inside the Kubernetes cluster.

The service type is:

```text
ClusterIP
```

---

# 🔐 Secrets Management

## Why Secrets Are Needed

The backend service uses OpenWeatherMap API.

The API key must not be stored in GitHub.

Instead, it is stored as a Kubernetes Secret.

---

## Example Secret File

A safe example file is stored in:

```text
examples/weather-backend-secret.example.env
```

Example content:

```env
OPENWEATHER_API_KEY=replace_with_your_openweathermap_api_key
```

This file is safe to commit because it does not contain a real API key.

---

## Real Secret File

Create a real local secret file from the example:

```powershell
copy examples\weather-backend-secret.example.env weather-backend-secret.env
```

Edit it:

```powershell
notepad weather-backend-secret.env
```

Set your real API key:

```env
OPENWEATHER_API_KEY=your_real_openweathermap_api_key
```

---

## `.gitignore`

The real secret file is excluded from Git.

Example:

```gitignore
# Real secrets - do not upload
*.env
.env
weather-backend-secret.env
weather-API.txt
local-secrets/

# Example env files are safe to upload
!*.example.env
!examples/*.example.env
```

This ensures that this file will not be uploaded to GitHub:

```text
weather-backend-secret.env
```

To verify that Git ignores the real secret file:

```powershell
git check-ignore -v weather-backend-secret.env
```

Expected result:

```text
.gitignore:5:weather-backend-secret.env weather-backend-secret.env
```

---

## Create Kubernetes Secret for Dev

Create the `dev` namespace if it does not exist:

```powershell
kubectl create namespace dev
```

Create the backend secret:

```powershell
kubectl create secret generic weather-backend-secret `
  --from-env-file=weather-backend-secret.env `
  -n dev
```

Verify:

```powershell
kubectl get secret weather-backend-secret -n dev
```

---

## Create Kubernetes Secret for Prod

Create the `prod` namespace if it does not exist:

```powershell
kubectl create namespace prod
```

Create the backend secret:

```powershell
kubectl create secret generic weather-backend-secret `
  --from-env-file=weather-backend-secret.env `
  -n prod
```

Verify:

```powershell
kubectl get secret weather-backend-secret -n prod
```

---

## Update Existing Secret

If the Secret already exists and the API key needs to be updated:

```powershell
kubectl delete secret weather-backend-secret -n dev
kubectl create secret generic weather-backend-secret `
  --from-env-file=weather-backend-secret.env `
  -n dev
```

For production:

```powershell
kubectl delete secret weather-backend-secret -n prod
kubectl create secret generic weather-backend-secret `
  --from-env-file=weather-backend-secret.env `
  -n prod
```

---

# 🧪 Helm Validation

Before deploying with ArgoCD, validate that Helm renders Kubernetes YAML correctly.

Run all commands from the repository root:

```powershell
cd C:\devop-proj\weather-helm-charts
```

---

## Backend Dev Validation

```powershell
helm template weather-backend-dev charts/weather-backend -f charts/weather-backend/values-dev.yaml
```

Expected output includes:

```yaml
namespace: dev
image: "harirs/weather-backend:dev"
```

---

## Frontend Dev Validation

```powershell
helm template weather-frontend-dev charts/weather-frontend -f charts/weather-frontend/values-dev.yaml
```

Expected output includes:

```yaml
namespace: dev
image: "harirs/weather-frontend:dev"
```

---

## Backend Prod Validation

```powershell
helm template weather-backend-prod charts/weather-backend -f charts/weather-backend/values-prod.yaml
```

Expected output includes:

```yaml
namespace: prod
image: "harirs/weather-backend:prod"
```

---

## Frontend Prod Validation

```powershell
helm template weather-frontend-prod charts/weather-frontend -f charts/weather-frontend/values-prod.yaml
```

Expected output includes:

```yaml
namespace: prod
image: "harirs/weather-frontend:prod"
```

---

## Check Only Replicas

For example, to check backend production replicas:

```powershell
helm template weather-backend-prod charts/weather-backend -f charts/weather-backend/values-prod.yaml | Select-String "replicas:"
```

Expected output:

```text
replicas: 1
```

---

# 🚢 ArgoCD Setup

## Install ArgoCD

Create the ArgoCD namespace:

```powershell
kubectl create namespace argocd
```

Install ArgoCD:

```powershell
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Check ArgoCD Pods:

```powershell
kubectl get pods -n argocd
```

Expected result:

```text
argocd-application-controller   Running
argocd-applicationset-controller Running
argocd-dex-server               Running
argocd-notifications-controller Running
argocd-redis                    Running
argocd-repo-server              Running
argocd-server                   Running
```

---

## Access ArgoCD UI

Use port-forward:

```powershell
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Open:

```text
https://localhost:8080
```

Get the initial admin password:

```powershell
kubectl -n argocd get secret argocd-initial-admin-secret `
  -o jsonpath="{.data.password}" | % { [System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($_)) }
```

Username:

```text
admin
```

---

# 🧩 ArgoCD Applications

This repository contains four ArgoCD Applications:

| Application | Chart Path | Values File | Namespace |
|---|---|---|---|
| `weather-backend-dev` | `charts/weather-backend` | `values-dev.yaml` | `dev` |
| `weather-frontend-dev` | `charts/weather-frontend` | `values-dev.yaml` | `dev` |
| `weather-backend-prod` | `charts/weather-backend` | `values-prod.yaml` | `prod` |
| `weather-frontend-prod` | `charts/weather-frontend` | `values-prod.yaml` | `prod` |

---

## `argocd/weather-backend-dev.yaml`

Deploys backend to the `dev` namespace.

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: weather-backend-dev
  namespace: argocd
spec:
  project: default

  source:
    repoURL: https://github.com/sagiv-Ha/weather-helm-charts.git
    targetRevision: main
    path: charts/weather-backend
    helm:
      valueFiles:
        - values-dev.yaml

  destination:
    server: https://kubernetes.default.svc
    namespace: dev

  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
```

---

## `argocd/weather-frontend-dev.yaml`

Deploys frontend to the `dev` namespace.

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: weather-frontend-dev
  namespace: argocd
spec:
  project: default

  source:
    repoURL: https://github.com/sagiv-Ha/weather-helm-charts.git
    targetRevision: main
    path: charts/weather-frontend
    helm:
      valueFiles:
        - values-dev.yaml

  destination:
    server: https://kubernetes.default.svc
    namespace: dev

  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
```

---

## `argocd/weather-backend-prod.yaml`

Deploys backend to the `prod` namespace.

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: weather-backend-prod
  namespace: argocd
spec:
  project: default

  source:
    repoURL: https://github.com/sagiv-Ha/weather-helm-charts.git
    targetRevision: main
    path: charts/weather-backend
    helm:
      valueFiles:
        - values-prod.yaml

  destination:
    server: https://kubernetes.default.svc
    namespace: prod

  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
```

---

## `argocd/weather-frontend-prod.yaml`

Deploys frontend to the `prod` namespace.

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: weather-frontend-prod
  namespace: argocd
spec:
  project: default

  source:
    repoURL: https://github.com/sagiv-Ha/weather-helm-charts.git
    targetRevision: main
    path: charts/weather-frontend
    helm:
      valueFiles:
        - values-prod.yaml

  destination:
    server: https://kubernetes.default.svc
    namespace: prod

  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
```

---

# ▶️ Deploy With ArgoCD

Run the following commands from the repository root.

## Apply Backend Dev

```powershell
kubectl apply -f argocd\weather-backend-dev.yaml
```

## Apply Frontend Dev

```powershell
kubectl apply -f argocd\weather-frontend-dev.yaml
```

## Apply Backend Prod

```powershell
kubectl apply -f argocd\weather-backend-prod.yaml
```

## Apply Frontend Prod

```powershell
kubectl apply -f argocd\weather-frontend-prod.yaml
```

---

# ✅ Verify ArgoCD Applications

Check all ArgoCD applications:

```powershell
kubectl get applications -n argocd
```

Expected output:

```text
NAME                    SYNC STATUS   HEALTH STATUS
weather-backend-dev     Synced        Healthy
weather-backend-prod    Synced        Healthy
weather-frontend-dev    Synced        Healthy
weather-frontend-prod   Synced        Healthy
```

---

# ✅ Verify Kubernetes Resources

Check `dev` environment:

```powershell
kubectl get all -n dev
```

Check `prod` environment:

```powershell
kubectl get all -n prod
```

Expected resources:

```text
weather-backend
weather-frontend
```

---

# 🧪 GitOps Test Performed

A GitOps test was performed by changing the backend production replicas from `1` to `2`.

## Step 1: Change replicas

File:

```text
charts/weather-backend/values-prod.yaml
```

Changed:

```yaml
replicas: 1
```

To:

```yaml
replicas: 2
```

## Step 2: Commit and push

```powershell
git add charts/weather-backend/values-prod.yaml
git commit -m "Scale backend prod replicas"
git push
```

## Step 3: ArgoCD detected the change

ArgoCD automatically synchronized the updated desired state.

## Step 4: Verify Kubernetes Deployment

```powershell
kubectl get deployment weather-backend -n prod
```

Expected result during the test:

```text
weather-backend   2/2
```

## Step 5: Restore replicas

The value was changed back to:

```yaml
replicas: 1
```

Then committed and pushed again:

```powershell
git add charts/weather-backend/values-prod.yaml
git commit -m "Restore backend prod replicas"
git push
```

ArgoCD synced the change and restored the Deployment to one replica.

This confirmed that the GitOps flow works correctly.

---

# 🔄 CI/CD Integration

The application repositories should update this Helm repository during CI/CD.

## Backend CI/CD Flow

```text
weather-backend repo
        ↓
GitHub Actions
        ↓
Docker build
        ↓
Docker push to Docker Hub
        ↓
Update charts/weather-backend/values-dev.yaml or values-prod.yaml
        ↓
Push change to weather-helm-charts
        ↓
ArgoCD syncs Kubernetes
```

## Frontend CI/CD Flow

```text
weather-frontend repo
        ↓
GitHub Actions
        ↓
Docker build
        ↓
Docker push to Docker Hub
        ↓
Update charts/weather-frontend/values-dev.yaml or values-prod.yaml
        ↓
Push change to weather-helm-charts
        ↓
ArgoCD syncs Kubernetes
```

---

# 🌿 Branch Strategy

| Branch | Environment | Action |
|---|---|---|
| `dev` | Development | Build image with `dev` tag and update `values-dev.yaml` |
| `master` / `main` | Production | Build image with `prod` tag and update `values-prod.yaml` |

---

# 🔎 Troubleshooting

## ArgoCD Application Not Showing

Check if ArgoCD is installed:

```powershell
kubectl get pods -n argocd
```

Check applications:

```powershell
kubectl get applications -n argocd
```

---

## Application Is Not Syncing

Force refresh:

```powershell
kubectl patch application weather-backend-prod -n argocd --type merge -p "{\"metadata\":{\"annotations\":{\"argocd.argoproj.io/refresh\":\"hard\"}}}"
```

Then check:

```powershell
kubectl get applications -n argocd
```

---

## Pods Are Not Starting

Check Pods:

```powershell
kubectl get pods -n dev
kubectl get pods -n prod
```

Describe a failing Pod:

```powershell
kubectl describe pod <pod-name> -n dev
```

Check logs:

```powershell
kubectl logs <pod-name> -n dev
```

---

## Backend Fails Because Secret Is Missing

Check if the Secret exists:

```powershell
kubectl get secret weather-backend-secret -n dev
kubectl get secret weather-backend-secret -n prod
```

If missing, create it:

```powershell
kubectl create secret generic weather-backend-secret `
  --from-env-file=weather-backend-secret.env `
  -n dev
```

And for production:

```powershell
kubectl create secret generic weather-backend-secret `
  --from-env-file=weather-backend-secret.env `
  -n prod
```

---

## Helm Output Looks Wrong

Render the chart locally:

```powershell
helm template weather-backend-prod charts/weather-backend -f charts/weather-backend/values-prod.yaml
```

Check replicas:

```powershell
helm template weather-backend-prod charts/weather-backend -f charts/weather-backend/values-prod.yaml | Select-String "replicas:"
```

---

# 🧼 Git Safety Checklist

Before pushing changes:

```powershell
git status
```

Make sure this file is **not** staged:

```text
weather-backend-secret.env
```

Check if it is ignored:

```powershell
git check-ignore -v weather-backend-secret.env
```

Expected result:

```text
.gitignore:5:weather-backend-secret.env weather-backend-secret.env
```

Commit safe files only:

```powershell
git add .
git commit -m "Update Helm GitOps configuration"
git push
```

---

# 📋 Final Status

The following setup has been completed:

| Component | Status |
|---|---|
| Backend Helm chart | ✅ Done |
| Frontend Helm chart | ✅ Done |
| Dev values files | ✅ Done |
| Prod values files | ✅ Done |
| Kubernetes Secret handling | ✅ Done |
| Secret example file | ✅ Done |
| `.gitignore` for real secrets | ✅ Done |
| ArgoCD installation | ✅ Done |
| Backend dev Application | ✅ Synced / Healthy |
| Frontend dev Application | ✅ Synced / Healthy |
| Backend prod Application | ✅ Synced / Healthy |
| Frontend prod Application | ✅ Synced / Healthy |
| GitOps replica test | ✅ Passed |

---

# 🧭 Related Repositories

| Repository | Purpose |
|---|---|
| `weather-backend` | Backend source code, Dockerfile, GitHub Actions |
| `weather-frontend` | Frontend source code, Dockerfile, GitHub Actions |
| `weather-helm-charts` | Helm charts and ArgoCD GitOps manifests |
| `weather-infra` / `weather-terraform` | Future infrastructure repository for Terraform and AWS |

---

# 🏁 Summary

This repository is responsible for the Kubernetes deployment layer of the Weather App.

It contains:

- Helm charts for backend and frontend.
- Separate configuration for `dev` and `prod`.
- ArgoCD Application manifests.
- Secret example file.
- GitOps deployment instructions.
- Validation and troubleshooting commands.

The final GitOps flow is:

```text
Code Change
   ↓
GitHub Actions
   ↓
Docker Image Build
   ↓
Docker Hub Push
   ↓
Helm Values Update
   ↓
Git Push to Helm Repo
   ↓
ArgoCD Auto Sync
   ↓
Kubernetes Deployment Updated
```

✅ The Weather App is now ready for GitOps-based deployment with Helm and ArgoCD.