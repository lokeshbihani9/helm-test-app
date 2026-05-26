# Helm Test App

A simple nginx application for testing Harness Helm deployments.

## Structure

```
charts/
  nginx-app/
    Chart.yaml          # Chart metadata
    values.yaml         # Default values
    templates/
      deployment.yaml   # Nginx deployment
      service.yaml      # Service
```

## Local Testing

Test the chart locally:

```bash
# Validate the chart
helm lint charts/nginx-app

# Dry run
helm install test-release charts/nginx-app --dry-run --debug

# Install locally
helm install test-release charts/nginx-app -n test --create-namespace

# Uninstall
helm uninstall test-release -n test
```

## Harness Configuration

### Service Configuration
- **Service Name**: nginx-app
- **Deployment Type**: Kubernetes (Helm)
- **Manifest Type**: Helm Chart
- **Store**: Git
- **Git Connector**: Select your GitHub connector
- **Branch**: main
- **Chart Path**: `charts/nginx-app`

### Infrastructure
- **Namespace**: `lbihani9-helm` (or your namespace)
- **Release Name**: `<+service.name>-<+env.name>`

### Reproducing the Bug

1. **Git Connector** should have delegate selector: `helm-delegate-ng`
2. **K8s Connector** can have any delegates with K8s access
3. **Canary Step** should have delegate selector: `helm-delegate-ng-2`

Expected: Canary deploy fails with "No eligible delegates" requiring both selectors.
