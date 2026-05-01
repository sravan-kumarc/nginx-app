Got it—this is a great move. Documenting this properly will **massively strengthen your profile**.

Below is a clean, professional **README.md** you can directly add to your repo.

---

# End-to-End GitOps CI/CD Pipeline (GitHub Actions + Argo CD + Helm) using NodeIP

## 📌 Overview

This project demonstrates a **production-style GitOps CI/CD pipeline** where:

* CI builds and pushes Docker images
* CI updates Helm configuration in Git
* Argo CD automatically deploys changes to Kubernetes

👉 No direct Kubernetes access from CI
👉 Git remains the **single source of truth**

---

## 🧱 Architecture

```
Developer → GitHub (nginx-app)
              ↓
        GitHub Actions (CI)
              ↓
     Docker Image (Docker Hub)
              ↓
     Update Helm values in Git
              ↓
 GitHub (argocd-helm-app repo)
              ↓
        Argo CD (CD)
              ↓
        Kubernetes Cluster
              ↓
            Browser
```

---

## 📦 Repositories Used

### 1. Application Repository (CI)

🔗 [https://github.com/sravan-kumarc/nginx-app/](https://github.com/sravan-kumarc/nginx-app/)

Contains:

* Dockerfile
* Application code (index.html)
* GitHub Actions workflow

---

### 2. Configuration Repository (CD)

🔗 [https://github.com/sravan-kumarc/argocd-helm-app/](https://github.com/sravan-kumarc/argocd-helm-app/)

Contains:

* Helm chart (`nginx-chart`)
* Environment-specific values
* GitOps configuration

---

## 🐳 Docker Setup

### Dockerfile

```dockerfile
FROM nginx:latest
COPY index.html /usr/share/nginx/html/index.html
```

---

## 🔐 GitHub Secrets

Configured in **nginx-app repo → Settings → Secrets**

```
DOCKER_USERNAME = <dockerhub-username>
DOCKER_PASSWORD = <dockerhub-access-token>
GH_TOKEN        = <github-personal-access-token>
```

---

## ⚙️ GitHub Actions CI Pipeline

📁 `.github/workflows/ci.yaml`

```yaml
name: CI Pipeline

on:
  push:
    branches:
      - main

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest

    steps:

    - name: Checkout code
      uses: actions/checkout@v3

    - name: Set image tag
      run: echo "IMAGE_TAG=${{ github.sha }}" >> $GITHUB_ENV

    - name: Login to Docker Hub
      run: echo "${{ secrets.DOCKER_PASSWORD }}" | docker login -u "${{ secrets.DOCKER_USERNAME }}" --password-stdin

    - name: Build Docker image
      run: |
        docker build -t ${{ secrets.DOCKER_USERNAME }}/nginx-gitops:${IMAGE_TAG} .

    - name: Push Docker image
      run: |
        docker push ${{ secrets.DOCKER_USERNAME }}/nginx-gitops:${IMAGE_TAG}

    - name: Update Helm values
      run: |
        git clone https://x-access-token:${{ secrets.GH_TOKEN }}@github.com/sravan-kumarc/argocd-helm-app.git
        cd argocd-helm-app

        sed -i "s/tag:.*/tag: ${IMAGE_TAG}/" values/values-dev.yaml

        git config user.name "github-actions"
        git config user.email "actions@github.com"

        git add .
        git commit -m "Update image tag to ${IMAGE_TAG}"
        git push
```

---

## 📊 Helm Configuration

📁 `values/values-dev.yaml`

```yaml
image:
  repository: sravankumar1006/nginx-gitops
  tag: <auto-updated-by-ci>
```

---

## 🚀 Argo CD Application (DEV)

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: nginx-dev
  namespace: argocd

spec:
  project: default

  source:
    repoURL: https://github.com/sravan-kumarc/argocd-helm-app
    targetRevision: main
    path: nginx-chart
    helm:
      valueFiles:
        - values/values-dev.yaml

  destination:
    server: https://kubernetes.default.svc
    namespace: dev

  syncPolicy:
    automated:
      prune: true
      selfHeal: true

    syncOptions:
      - CreateNamespace=true
```

---

## 🔄 CI/CD Flow Explained

### Step 1 — Code Change

Developer updates application code:

```html
<h1>GitOps Demo - Version 2</h1>
```

---

### Step 2 — CI Triggered

GitHub Actions pipeline runs:

* Builds Docker image
* Tags with commit SHA
* Pushes to Docker Hub

---

### Step 3 — GitOps Update

Pipeline updates Helm values:

```yaml
tag: <new-commit-sha>
```

Commit pushed to:
👉 `argocd-helm-app` repo

---

### Step 4 — Argo CD Sync

Argo CD detects Git change:

* Marks app OutOfSync
* Auto-syncs
* Deploys new version

---

### Step 5 — Kubernetes Update

* New pods created
* Old pods terminated
* Zero manual intervention

---

### Step 6 — Verification

Application accessible via:

```
http://<EC2-IP>:8081
```

---

## ✅ Result

* Version updated in browser automatically
* No manual deployment
* Full GitOps workflow achieved

---

## 🔥 Key Principles Implemented

* Git as Single Source of Truth
* CI updates Git, not Kubernetes
* Declarative deployments (Helm)
* Automated sync (Argo CD)
* Immutable image tagging (SHA-based)

---

## ⚠️ Common Pitfalls Avoided

* ❌ Using `latest` image tag
* ❌ Direct `kubectl apply` from CI
* ❌ Hardcoded credentials
* ❌ Manual deployments

---

## 🎯 Outcome

Successfully implemented a **production-style GitOps pipeline** integrating:

* GitHub Actions (CI)
* Docker Hub (Registry)
* Helm (Configuration)
* Argo CD (CD)
* Kubernetes (Runtime)

---

## 🚀 Next Steps

* Replace port-forward with **Ingress**
* Add **multi-environment promotion (dev → uat → prod)**
* Implement **AppProject governance**
* Add **observability (Prometheus + Grafana)**

---

