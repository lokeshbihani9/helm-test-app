# Quick Start Guide

## Your Environment
- **Delegates**: 
  - `helm-delegate-ng` (for GitHub access)
  - `helm-delegate-ng-2` (for K8s access)
- **K8s Namespace**: `lbihani9-helm`
- **Deployment Type**: Helm Native
- **Platform**: Kubernetes

## 1. Push to GitHub (2 minutes)

```bash
cd /Users/lbihani9/dev/cd-play/repos/helm-test-app

# If using GitHub CLI
gh repo create lokeshbihani9/helm-test-app --public --source=. --remote=origin --push

# OR manually create repo at github.com, then:
git remote add origin git@github.com:lokeshbihani9/helm-test-app.git
git push -u origin main
```

## 2. Harness Configuration (10 minutes)

### Git Connector
```
Name: github-helm-test
URL: https://github.com/lokeshbihani9/helm-test-app
Selector: helm-delegate-ng  ⚠️ CRITICAL
```

### Service
```
Name: nginx-app
Type: Kubernetes
Manifest: Helm Chart from Git
  └─ Connector: github-helm-test
  └─ Branch: main
  └─ Path: charts/nginx-app
```

### Environment & Infrastructure
```
Environment: dev
Infrastructure: k8s-dev
  └─ Namespace: lbihani9-helm
  └─ K8s Connector: (your existing K8s connector)
```

### Pipeline (Bug Reproduction)
```
Pipeline: helm-canary-bug-repro
Stage: Deploy
  └─ Service: nginx-app
  └─ Environment: dev
  └─ Execution:
      └─ K8s Canary Deploy
          ├─ Delegate Selector: helm-delegate-ng-2  ⚠️ CRITICAL
          └─ Instance Count: 1
```

## 3. Run & Observe

**Expected Failure**:
```
Task 1: Fetch Files ✅
  → Delegate: helm-delegate-ng

Task 2: Helm Canary Deploy ❌
  → Error: No eligible delegates
  → Required: [helm-delegate-ng, helm-delegate-ng-2]
  → Available:
     - helm-delegate-ng: Missing helm-delegate-ng-2
     - helm-delegate-ng-2: Missing helm-delegate-ng
```

## 4. Comparison Test (Optional)

Create Rolling pipeline WITHOUT step-level delegate selector:
- Should succeed using only `helm-delegate-ng`
- Proves the bug is specific to selector combination

## Key Points

✅ **Git connector selector**: Forces Git fetch to specific delegate  
✅ **Step-level selector**: Forces Helm execution to different delegate  
❌ **Bug**: Helm task requires BOTH selectors (AND logic)  
💡 **Root cause**: Git capabilities leak into Helm command task even after files are fetched

## Verification Checklist

- [ ] Repository pushed to GitHub
- [ ] Git connector created with `helm-delegate-ng` selector
- [ ] Service configured with Helm chart from Git
- [ ] Infrastructure uses `lbihani9-helm` namespace
- [ ] Canary step has `helm-delegate-ng-2` selector
- [ ] Delegates are running and have distinct tags
- [ ] Pipeline execution shows delegate selection failure

## Clean Up

```bash
# Delete Helm releases
helm list -n lbihani9-helm
helm uninstall nginx-app-dev -n lbihani9-helm

# Delete Kubernetes resources if any remain
kubectl delete all -l app=nginx-app -n lbihani9-helm
```
