# BoardGame GitOps Repository

This repository contains Kubernetes deployment manifests for the BoardGame application using GitOps principles with ArgoCD.

## Repository Structure
```
boardgame-gitops/
├── base/                           # Helm chart base (shared)
│   ├── Chart.yaml
│   ├── values.yaml                # Base configuration
│   └── templates/
│       ├── deployment.yaml
│       ├── service.yaml
│       ├── ingress.yaml
│       └── _helpers.tpl
├── apps/                           # Environment overlays
│   ├── dev/
│   │   └── values-override.yaml   # Dev-specific config
│   └── prod/
│       └── values-override.yaml   # Prod-specific config
└── sealed-secrets/                 # Encrypted secrets
    └── harbor-registry-sealed.yaml
```

## Environments

- **Dev**: `boardgame-dev.app.thweb.click` (1 replica, no SSL)
- **Prod**: `boardgame.app.thweb.click` (3 replicas, SSL enabled)

## ArgoCD Applications

- `boardgame-dev`: Syncs from `apps/dev/`
- `boardgame-prod`: Syncs from `apps/prod/`

## Workflow

1. Jenkins builds app → pushes image to Harbor
2. Jenkins updates `image.tag` in `apps/{env}/values-override.yaml`
3. Jenkins commits & pushes to this repo
4. ArgoCD detects change → syncs to cluster (within 3min)
5. Kubernetes performs rolling update

## Manual Deployment
```bash
# Dev
helm upgrade --install boardgame-dev ./base \
  -f apps/dev/values-override.yaml \
  -n boardgame-dev --create-namespace

# Prod
helm upgrade --install boardgame-prod ./base \
  -f apps/prod/values-override.yaml \
  -n boardgame --create-namespace
```

## Rollback
```bash
# Git-based rollback
git revert <commit-sha>
git push
# ArgoCD will auto-sync

# Or manual helm rollback
helm rollback boardgame-dev -n boardgame-dev
```
