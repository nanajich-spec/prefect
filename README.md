# Prefect GitOps Repository

This repository contains Kubernetes manifests for deploying Prefect orchestration platform, managed by ArgoCD.

## Overview

Prefect is a modern workflow orchestration tool deployed on Kubernetes with the following components:
- **Prefect Server**: API and UI backend
- **Prefect Agent**: Task executor worker
- **PostgreSQL**: Metadata storage database

## Repository Structure

```
.
├── README.md                      # This file
├── namespace.yaml                 # Prefect namespace
├── rbac.yaml                      # RBAC resources (ServiceAccount, Role, RoleBinding)
├── postgres-pvc.yaml              # PostgreSQL persistent volume claim
├── postgres-deployment.yaml       # PostgreSQL database deployment
├── server-deployment.yaml         # Prefect server deployment
└── agent-deployment.yaml          # Prefect agent deployment
```

## Deployment Order

ArgoCD will apply resources in the following order:
1. Namespace
2. RBAC (ServiceAccount, Role, RoleBinding)
3. PostgreSQL PVC
4. PostgreSQL Deployment & Service
5. Prefect Server Deployment & Service
6. Prefect Agent Deployment

## ArgoCD Application

This repository is deployed as an ArgoCD Application with auto-sync enabled.

### Application Manifest

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: prefect
  namespace: argocd
spec:
  project: default
  source:
    repoURL: <your-github-repo-url>
    targetRevision: HEAD
    path: .
  destination:
    server: https://kubernetes.default.svc
    namespace: prefect
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
    - CreateNamespace=true
```

## Access Information

After deployment:
- **Prefect UI**: Port-forward with `kubectl port-forward -n prefect svc/prefect-server 4200:4200`
- **Access UI**: http://localhost:4200
- **API Endpoint**: http://localhost:4200/api

## Development Workflow

1. Make changes to manifests in this repository
2. Commit and push to GitHub
3. ArgoCD automatically detects changes
4. ArgoCD syncs and applies updates to the cluster
5. Monitor changes in ArgoCD UI: https://localhost:8082

## Resources

- **Prefect Version**: 2.20.25
- **PostgreSQL Version**: 15-alpine
- **Storage**: 5Gi PVC for PostgreSQL data

## Performance

Prefect scheduling performance:
- **Scheduling rate**: 168+ tasks/second
- **Task overhead**: 5-10ms per task
- **Throughput**: ~10 tasks/second end-to-end

## Scaling

To scale components, modify the `replicas` field:
- Prefect Server: Currently 1 replica (recommended for single-node)
- Prefect Agent: Currently 1 replica (can scale for higher throughput)
- PostgreSQL: 1 replica (StatefulSet for production)

## Monitoring

Check pod status:
```bash
kubectl get pods -n prefect
kubectl logs -n prefect -l app=prefect-server -f
kubectl logs -n prefect -l app=prefect-agent -f
```

## Managed by ArgoCD

This application is managed by ArgoCD. Direct kubectl changes may be overwritten.
Use GitOps workflow: commit to this repo → ArgoCD syncs → cluster updated.
