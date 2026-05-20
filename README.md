# 🚀 Transport - CI/CD Setup Guide

## Branch Strategy
```
dev  →  (auto deploy)  →  staging  →  (manual approve)  →  main (production)
```

## 📁 File Structure
```
transport/
├── .github/
│   └── workflows/
│       ├── dev-to-staging.yml       # dev push → staging auto deploy
│       └── staging-to-production.yml # main push → production deploy
├── k8s/
│   ├── staging/
│   │   └── deployment.yml
│   └── production/
│       └── deployment.yml
├── src/
│   └── index.js
├── Dockerfile                       # Dev/Staging image
├── Dockerfile.prod                  # Production optimized image
├── package.json
└── .gitignore
```

---

## ⚙️ GitHub Secrets Setup

Go to: **GitHub → transport- repo → Settings → Secrets → Actions**

Add these secrets:

| Secret Name | Value |
|---|---|
| `KUBECONFIG_STAGING` | base64 encoded kubeconfig for staging cluster |
| `KUBECONFIG_PRODUCTION` | base64 encoded kubeconfig for production cluster |

### How to encode kubeconfig:
```bash
cat ~/.kube/config | base64 -w 0
```

---

## 🔐 GitHub Environments Setup (Manual Approval)

Go to: **GitHub → Settings → Environments**

1. Create environment: `staging`
2. Create environment: `production`
   - ✅ Add **Required reviewers** (your GitHub username)
   - This will pause the pipeline and ask for approval before deploying to production

---

## 🐳 GHCR Pull Secret for Kubernetes

Run this on your K8s cluster:

```bash
# Create namespace
kubectl create namespace staging
kubectl create namespace production

# Create image pull secret
kubectl create secret docker-registry ghcr-secret \
  --docker-server=ghcr.io \
  --docker-username=kannakamalesh1-cloud \
  --docker-password=YOUR_GITHUB_PAT_TOKEN \
  --docker-email=your@email.com \
  -n staging

kubectl create secret docker-registry ghcr-secret \
  --docker-server=ghcr.io \
  --docker-username=kannakamalesh1-cloud \
  --docker-password=YOUR_GITHUB_PAT_TOKEN \
  --docker-email=your@email.com \
  -n production
```

---

## 🚀 First Time Git Setup

```bash
git clone https://github.com/kannakamalesh1-cloud/transport-.git
cd transport-

# Create branches
git checkout -b dev
git add .
git commit -m "Initial setup: Add CI/CD pipeline, Dockerfiles, K8s configs"
git push origin dev

git checkout -b staging
git push origin staging

git checkout main
git push origin main
```

---

## 🔄 Flow Summary

```
You push to dev
    ↓
GitHub Actions builds Docker image
    ↓
Pushes to ghcr.io
    ↓
Auto deploys to K8s staging namespace
    ↓
You verify staging.transport.in
    ↓
Merge dev → main (PR)
    ↓
GitHub asks for MANUAL APPROVAL
    ↓
After approval → Production deploys to transport.in
```
