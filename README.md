# K3s GitOps Demo

## Overview

This repository demonstrates a complete GitOps workflow using Kubernetes (K3s), ArgoCD, and Git-based deployment strategies. It provides a foundation for implementing GitOps best practices in your infrastructure.

## What is GitOps?

GitOps is a way of managing infrastructure and applications where:
- Git is the single source of truth
- Changes are tracked via Git commits
- Automated systems sync desired state from Git to actual infrastructure
- All changes are auditable and reversible

## Architecture

```
Git Repository (GitHub)
        ↓
   [Manifests]
        ↓
  [ArgoCD]
   ↙  ↓  ↘
[App1] [App2] [App3]
        ↓
   [K3s Cluster]
        ↓
   [Workloads]
```

## Tech Stack

| Component | Version |
|-----------|----------|
| Kubernetes | K3s 1.27+ |
| GitOps | ArgoCD 2.8+ |
| Cluster Autoscaling | Karpenter (optional) |
| Monitoring | Prometheus + Grafana |
| Logging | Loki |
| Ingress | Traefik |
| Storage | Local Volumes, NFS |

## Prerequisites

- Docker or container runtime
- kubectl >= 1.24
- Helm >= 3.12
- Git
- Basic Kubernetes knowledge

## Quick Start

### 1. Setup K3s Cluster

```bash
# Install K3s on Linux
curl -sfL https://get.k3s.io | sh -

# Verify installation
kubectl cluster-info
kubectl get nodes
```

### 2. Install ArgoCD

```bash
# Create ArgoCD namespace
kubectl create namespace argocd

# Install ArgoCD
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Wait for deployment
kubectl wait -n argocd --for=condition=available --timeout=300s deployment/argocd-server

# Port forward to access UI
kubectl port-forward svc/argocd-server -n argocd 8080:443
# Access at: https://localhost:8080
```

### 3. Get Initial ArgoCD Password

```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

### 4. Configure Git Repository

```bash
# Clone this repository
git clone https://github.com/BODAPATI88/k8s-gitops-demo.git
cd k8s-gitops-demo

# Create your own repository from this template
# Then update ArgoCD to point to your repository
```

### 5. Deploy Application via ArgoCD

```bash
# Create ArgoCD Application
kubectl apply -f argocd/application.yaml

# Check application status
kubectl get application -n argocd

# View sync status
kubectl describe application my-app -n argocd
```

## Project Structure

```
k8s-gitops-demo/
├── README.md                    # This file
├── .gitignore                   # Git ignore rules
│
├── argocd/
│   ├── application.yaml         # ArgoCD Application resource
│   ├── appproject.yaml          # AppProject for RBAC
│   └── repository.yaml          # Git repository config
│
├── apps/
│   ├── nginx-demo/
│   │   ├── deployment.yaml      # Nginx deployment
│   │   ├── service.yaml         # Service definition
│   │   ├── ingress.yaml         # Ingress route
│   │   └── kustomization.yaml   # Kustomize overlay
│   │
│   ├── app-2/
│   │   ├── helm/                # Helm chart
│   │   └── values.yaml
│   │
│   └── app-3/
│       └── manifests/
│
├── infrastructure/
│   ├── namespaces.yaml          # Namespace definitions
│   ├── rbac.yaml                # RBAC policies
│   ├── network-policies.yaml    # Network segmentation
│   └── storage.yaml             # Storage classes
│
├── monitoring/
│   ├── prometheus/              # Prometheus config
│   ├── grafana/                 # Grafana dashboards
│   └── loki/                    # Log aggregation
│
├── addons/
│   ├── cert-manager.yaml        # SSL certificates
│   ├── external-dns.yaml        # DNS automation
│   └── sealed-secrets.yaml      # Secret encryption
│
└── docs/
    ├── ARCHITECTURE.md          # Architecture details
    ├── DEPLOYMENT.md            # Deployment guide
    └── TROUBLESHOOTING.md       # Common issues
```

## Key Concepts

### Declarative Configuration

All infrastructure state is declared in YAML:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: app
    image: nginx:latest
    ports:
    - containerPort: 80
```

### GitOps Workflow

1. **Developer** makes changes to Git
2. **Git** triggers webhook to ArgoCD
3. **ArgoCD** detects changes
4. **ArgoCD** syncs cluster to match Git
5. **Kubernetes** reconciles desired state

### Kustomize Overlays

Manage multiple environments:

```
apps/nginx-demo/
├── base/
│   ├── deployment.yaml
│   └── kustomization.yaml
└── overlays/
    ├── dev/
    │   └── kustomization.yaml    # 1 replica, dev resources
    ├── staging/
    │   └── kustomization.yaml    # 2 replicas, staging resources
    └── prod/
        └── kustomization.yaml    # 3 replicas, prod resources
```

## Usage

### Deploy Application

```bash
# Method 1: Using kubectl
kubectl apply -f apps/nginx-demo/

# Method 2: Using ArgoCD
kubectl apply -f argocd/application.yaml

# Method 3: Using Helm
helm install my-release ./apps/app-2/helm/
```

### Update Application

```bash
# Edit manifest in Git
git checkout -b feature/update-replicas
# Edit apps/nginx-demo/deployment.yaml
replicas: 3

# Commit and push
git add apps/nginx-demo/deployment.yaml
git commit -m "feat: scale nginx to 3 replicas"
git push origin feature/update-replicas

# Create Pull Request
# After merge to main, ArgoCD automatically syncs
```

### Monitor Applications

```bash
# Check deployment status
kubectl get deployments -n default
kubectl describe deployment nginx-demo

# View logs
kubectl logs -n default deployment/nginx-demo

# Check events
kubectl get events -n default

# Monitor via ArgoCD UI
kubectl port-forward svc/argocd-server -n argocd 8080:443
# Open https://localhost:8080
```

## Best Practices

### 1. Repository Structure
- Keep manifests organized by application/service
- Use consistent naming conventions
- Document dependencies

### 2. Version Control
```bash
# Always work in feature branches
git checkout -b feature/add-new-deployment

# Write meaningful commit messages
git commit -m "feat: add redis cache service"

# Require PR reviews before merging
```

### 3. Security
```bash
# Use sealed-secrets for sensitive data
echo -n mypassword | kubectl create secret generic mysecret \
  --dry-run=client --from-file=/dev/stdin -o yaml | \
  kubeseal -f -

# Never commit secrets to Git
# Always use RBAC
# Enable network policies
```

### 4. Testing
```bash
# Test manifests locally
kubectl apply -f apps/nginx-demo/ --dry-run=client

# Validate YAML
kubectl apply -f apps/nginx-demo/ -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: test
EOF

# Use kubeval for strict validation
kubeval apps/nginx-demo/
```

## Troubleshooting

### Application not syncing

```bash
# Check ArgoCD Application status
kubectl describe application my-app -n argocd

# View ArgoCD server logs
kubectl logs -n argocd deployment/argocd-server

# Manually sync
kubectl patch application my-app -n argocd \
  -p '{"metadata":{"labels":{"argocd.argoproj.io/compare-result":"Unknown"}}}'
```

### Pod not starting

```bash
# Check pod status
kubectl describe pod <pod-name>

# View logs
kubectl logs <pod-name>
kubectl logs <pod-name> --previous  # For crashed pods

# Check events
kubectl get events --sort-by='.lastTimestamp'
```

### Resource conflicts

```bash
# Check for duplicate resources
kubectl get deployment nginx-demo -o yaml

# Delete conflicting resource
kubectl delete deployment nginx-demo

# Let ArgoCD recreate it
```

## Advanced Topics

### Multi-cluster GitOps

```yaml
# Single Git repository, multiple clusters
argocd/
├── applications/
│   ├── cluster-1.yaml  # Applications for cluster 1
│   └── cluster-2.yaml  # Applications for cluster 2
└── infrastructure/
    ├── cluster-1/      # Infra for cluster 1
    └── cluster-2/      # Infra for cluster 2
```

### Progressive Delivery

Integrate with Flagger for canary deployments:

```yaml
apiVersion: flagger.app/v1beta1
kind: Canary
metadata:
  name: nginx
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx
  progressDeadlineSeconds: 60
  service:
    port: 80
  analysis:
    interval: 1m
    threshold: 5
    metrics:
    - name: request-success-rate
      thresholdRange:
        min: 99
      interval: 1m
```

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

## Resources

- [ArgoCD Documentation](https://argo-cd.readthedocs.io/)
- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [K3s Documentation](https://docs.k3s.io/)
- [GitOps Best Practices](https://www.weave.works/blog/gitops-operations-by-pull-request)

## License

MIT License - See [LICENSE](LICENSE) for details.

## Support

For questions and issues:
- Create an issue on GitHub
- Check [TROUBLESHOOTING.md](docs/TROUBLESHOOTING.md)
- Review [ARCHITECTURE.md](docs/ARCHITECTURE.md)
