# Setup Instructions

## Step 1: Create GitHub Repository

```bash
# Navigate to the repo
cd /Users/lbihani9/dev/cd-play/repos/helm-test-app

# Create a new repository on GitHub
# Option A: Using GitHub CLI
gh repo create lokeshbihani9/helm-test-app --public --source=. --remote=origin --push

# Option B: Manual (if gh not available)
# 1. Go to https://github.com/new
# 2. Create repository: helm-test-app
# 3. Then run:
git remote add origin git@github.com:lokeshbihani9/helm-test-app.git
git branch -M main
git push -u origin main
```

## Step 2: Configure Harness

### A. Create Git Connector

1. **Navigate to**: Project Settings → Connectors → + New Connector → GitHub
2. **Configure**:
   - Name: `github-helm-test`
   - URL Type: `Repository`
   - Connection Type: `SSH` or `HTTPS`
   - URL: `https://github.com/lokeshbihani9/helm-test-app`
3. **⚠️ CRITICAL - Delegates Setup**:
   - Enable: `Connect through Harness Delegate`
   - Selectors: `helm-delegate-ng`
   - This forces Git fetch to use only this delegate
4. **Test & Save**

### B. Create Service

1. **Navigate to**: Services → + New Service
2. **Configure**:
   - Name: `nginx-app`
   - Service Type: `Kubernetes`
3. **Add Manifest**:
   - Click: `+ Add Manifest`
   - Type: `Helm Chart`
   - Store: `Git`
   - Connector: `github-helm-test` (from step A)
   - Manifest Identifier: `nginx-helm`
   - Git Fetch Type: `Latest from Branch`
   - Branch: `main`
   - Chart Path: `charts/nginx-app`
   - Helm Version: `Version 3`
4. **Save**

### C. Create Environment & Infrastructure

1. **Navigate to**: Environments → + New Environment
2. **Configure**:
   - Name: `dev`
   - Type: `Pre-Production`
3. **Add Infrastructure**:
   - Name: `k8s-dev`
   - Deployment Type: `Kubernetes`
   - Type: `Kubernetes`
   - Connector: Your K8s connector (connected via `helm-delegate-ng-2`)
   - Namespace: `lbihani9-helm`
   - Release Name: `<+service.name>-<+env.name>`

### D. Create Pipeline - Canary Deploy (Will Fail - Bug Reproduction)

1. **Navigate to**: Pipelines → + Create Pipeline
2. **Name**: `helm-canary-bug-repro`
3. **Add Stage**:
   - Type: `Deploy`
   - Name: `Deploy Canary`
   - Service: `nginx-app`
   - Environment: `dev`
   - Infrastructure: `k8s-dev`
4. **Execution**:
   - Add Step: `Helm Deployment` → `K8s Canary Deploy`
   - Step Name: `Canary Deploy`
   - **⚠️ CRITICAL - Advanced → Delegate Selector**:
     - Add: `helm-delegate-ng-2`
   - Instance Selection:
     - Type: `Count`
     - Count: `1`
5. **Save**

### E. Create Pipeline - Rolling Deploy (Should Work - Comparison)

1. **Duplicate previous pipeline** OR create new
2. **Name**: `helm-rolling-works`
3. **Execution**:
   - Replace Canary step with: `Helm Deploy` (Rolling)
   - **Do NOT add delegate selector at step level**
   - Or add the SAME selector as Git connector: `helm-delegate-ng`
4. **Save**

## Step 3: Reproduce the Bug

### Expected Results:

**Canary Pipeline (helm-canary-bug-repro)**:
- ✅ Fetch Files task: SUCCESS (uses `helm-delegate-ng`)
- ❌ Helm Canary Deploy task: FAILED
  - Error: "No eligible delegates could perform this task"
  - Required selectors: `[helm-delegate-ng, helm-delegate-ng-2]`
  - `helm-delegate-ng`: Missing `helm-delegate-ng-2` ❌
  - `helm-delegate-ng-2`: Missing `helm-delegate-ng` ❌

**Rolling Pipeline (helm-rolling-works)**:
- ✅ Fetch Files task: SUCCESS (uses `helm-delegate-ng`)
- ✅ Helm Deploy task: SUCCESS (uses `helm-delegate-ng` - no step selector added)

## Step 4: Verify Delegate Selection Logs

1. Click on failed Canary Deploy step
2. Go to "Delegate Selection" tab
3. You'll see both selectors required together (AND logic)
4. Check that no single delegate has both tags

## Troubleshooting

### If Canary succeeds unexpectedly:
- Verify Git connector has selector: `helm-delegate-ng`
- Verify Canary step has selector: `helm-delegate-ng-2`
- Verify delegates don't have overlapping tags

### If both fail:
- Check delegate connectivity
- Verify namespace exists: `kubectl get ns lbihani9-helm`
- Check K8s connector authentication

### If both succeed:
- One delegate might have both tags accidentally
- Check: Project Settings → Delegates → View delegates
- Ensure delegates have distinct, non-overlapping tags
