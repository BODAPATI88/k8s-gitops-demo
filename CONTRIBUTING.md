# Contributing to k8s-gitops-demo

Thank you for your interest in contributing to this GitOps demonstration!

## Code of Conduct

Please be respectful and professional in all interactions.

## Getting Started

### Prerequisites
- K3s cluster or local Kubernetes (Docker Desktop, Minikube)
- kubectl >= 1.24
- Git
- Basic Kubernetes knowledge

### Setup

```bash
# Clone repository
git clone https://github.com/BODAPATI88/k8s-gitops-demo.git
cd k8s-gitops-demo

# Create feature branch
git checkout -b feature/your-feature
```

## Making Changes

### 1. Application Changes

```bash
# Edit manifest
vim apps/nginx-demo/deployment.yaml

# Validate YAML
kubectl apply -f apps/nginx-demo/ --dry-run=client

# Test locally
kubectl apply -f apps/nginx-demo/
kubectl get pods
kubectl delete -f apps/nginx-demo/
```

### 2. Infrastructure Changes

```bash
# Edit infrastructure files
vim infrastructure/namespace.yaml

# Validate
kubectl apply -f infrastructure/ --dry-run=client

# Test
kubectl apply -f infrastructure/
```

### 3. ArgoCD Configuration

```bash
# Edit ArgoCD settings
vim argocd/application.yaml

# Validate Application resource
kubectl apply -f argocd/application.yaml --dry-run=client
```

## Commit Message Guidelines

Use conventional commits:

```
feat(apps): add redis deployment
fix(nginx): correct service port
docs: update deployment guide
refactor(argocd): simplify application config
```

## Testing

### Validate Manifests

```bash
# Check YAML syntax
kubectl apply -f . --dry-run=client --recursive

# Use kubeval
kubeval $(find . -name '*.yaml' -o -name '*.yml')

# Use kubesec for security
kubesec scan apps/nginx-demo/deployment.yaml
```

### Test Deployment

```bash
# Deploy to test cluster
kubectl apply -f .

# Verify resources
kubectl get deployments,services,ingresses

# Check logs
kubectl logs -l app=nginx-demo

# Cleanup
kubectl delete -f .
```

## Pull Request Process

### Before Submitting
1. Validate all manifests
2. Test deployment locally
3. Update documentation
4. Follow naming conventions
5. Ensure no hardcoded values

### PR Template

```markdown
## Description
Brief description of changes

## Type of Change
- [ ] New application
- [ ] Configuration update
- [ ] Infrastructure change
- [ ] Documentation

## Testing
How have these changes been tested?

## Checklist
- [ ] Manifests validated
- [ ] Tested locally
- [ ] Documentation updated
- [ ] No hardcoded values

## Related Issues
Closes #123
```

## Documentation

Update relevant documentation:
- README.md for overview changes
- docs/ARCHITECTURE.md for structural changes
- docs/DEPLOYMENT.md for deployment changes
- CONTRIBUTING.md for process changes

## Questions?

Feel free to open an issue or start a discussion!
