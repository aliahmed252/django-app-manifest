# django-app-manifest: GitOps Manifest Repository

## Overview
This repository serves as the **Source of Truth** for the deployment state of the `django-meal-rator` application. It contains all Kubernetes manifests managed via **Kustomize** and is monitored by **ArgoCD** for automated synchronization to AWS EKS.

## GitOps Relationship
This project uses a split-repository architecture to separate application logic from infrastructure state:
1.  **[django-meal-rator](https://github.com/aliahmed252/django-meal-rator)** (App Repo): Contains Django source code, Terraform (IaC), and GitHub Actions pipelines.
2.  **django-app-manifest** (This Repo): Contains the desired state of the Kubernetes cluster.

### The Automation Loop
1. **Build**: GitHub Actions in the App Repo builds a Docker image and pushes it to Docker Hub.
2. **Update**: The pipeline automatically updates the `newTag` in `apps/prod/kustomization.yaml` in this repository.
3. **Sync**: ArgoCD detects the commit and applies the changes to the **AWS EKS** cluster.

## Technical Stack
*   **Configuration Management**: Kustomize
*   **Continuous Delivery**: ArgoCD
*   **Orchestration**: Kubernetes (AWS EKS)

## Repository Structure
The manifests use Kustomize overlays to manage different environments:
```
django-app-manifest/
├── apps/
│   ├── base/               # Common resources (Deployment, Service, HPA)
│   ├── dev/                # Development environment overrides
│   ├── staging/            # Staging environment overrides
│   └── prod/               # Production environment overrides
│       ├── kustomization.yaml  # Main config (Target for CI image updates)
│       ├── patch-env.yaml      # Production-specific environment variables
│       └── patch-replicas.yaml # Production-specific scaling (e.g., 3+ replicas)
```

## Operational Commands

### View Generated Manifests
To see exactly what will be applied to the cluster for production:
```bash
kustomize build apps/prod
```

### Manual Sync
If you need to force a synchronization via the ArgoCD CLI:
```bash
argocd app sync malerator-prod
```

### Troubleshooting Pods
```bash
# List production pods
kubectl get pods -n malerator-prod

# Check deployment status
kubectl describe deployment malerator-app -n malerator-prod
```
