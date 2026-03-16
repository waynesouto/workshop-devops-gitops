# Workshop DevOps GitOps

A GitOps-oriented Kubernetes deployment repository for a sample application consisting of a frontend and backend service. This project is intended for workshops and learning GitOps practices using Argo CD and Kustomize.

## Overview

This repository holds the declarative Kubernetes manifests and Argo CD application definition used to deploy and manage the application on a Kubernetes cluster. Changes pushed to the repository are automatically reconciled by Argo CD, keeping the cluster state in sync with the desired configuration.

## Architecture

- **Argo CD**: Continuous deployment from this Git repository; syncs and self-heals the cluster based on the manifests defined here.
- **Kustomize**: Manifests are organized and parameterized with Kustomize (image tags, base resources).
- **Applications**: Two components are deployed:
  - **Backend**: Deployment and NodePort Service (port 30000).
  - **Frontend**: Deployment and NodePort Service (port 30001).

Container images are stored in Amazon ECR in the `us-east-1` region.

## Repository Structure

```
.
├── argocd/
│   └── application.yml    # Argo CD Application pointing to this repo
├── backend/
│   ├── deploy.yml         # Backend Deployment
│   └── service.yml        # Backend NodePort Service (30000)
├── frontend/
│   ├── deploy.yml         # Frontend Deployment
│   └── service.yml        # Frontend NodePort Service (30001)
├── kustomization.yml      # Kustomize root (resources + image tags)
└── README.md
```

## Prerequisites

- A Kubernetes cluster with Argo CD installed.
- `kubectl` configured to access the cluster.
- (Optional) `kustomize` or `kubectl build` for local preview.

## Deployment

### Via Argo CD

1. Ensure the Argo CD Application is created from `argocd/application.yml` (e.g. by applying it or by configuring Argo CD to use this repo).
2. Argo CD will use the root of this repository as the application path and Kustomize for building manifests.
3. With `syncPolicy.automated.selfHeal: true`, Argo CD will automatically sync and correct drift.

The Application targets the `default` namespace and the in-cluster API server (`https://kubernetes.default.svc`).

### Local Preview

To render the final manifests locally without applying:

```bash
kubectl kustomize .
```

To apply directly to the cluster (without Argo CD):

```bash
kubectl apply -k .
```

## Configuration

### Image Tags

Image names and tags are set in `kustomization.yml` under the `images` section. To promote a new version, update the `newTag` for the backend and/or frontend image and commit. Argo CD will pick up the change and sync the cluster.

Example (conceptual): change `newTag: v1.0` to `newTag: v1.1` for the desired image(s).

### Service Ports

- Backend: NodePort `30000`, container port `80`.
- Frontend: NodePort `30001`, container port `80`.

To change ports, edit the corresponding `service.yml` in `backend/` or `frontend/`.

## License

See the repository license file for terms of use.
