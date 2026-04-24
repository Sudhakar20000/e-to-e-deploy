# 🔄 End-to-End Deployment Pipeline (GitOps with Jenkins + ArgoCD + Kubernetes)

This repository is the **CD (Continuous Delivery) side** of a complete end-to-end DevOps pipeline. It holds the Kubernetes deployment manifests and a Jenkins pipeline that automatically updates those manifests whenever a new Docker image is built by the CI pipeline. ArgoCD then picks up the changes and deploys them to a Kubernetes cluster — no manual steps needed.

---

## 🧠 How It Works (The Big Picture)

```
CI Pipeline (another repo)
    → Builds Docker image
    → Pushes to Docker Hub with a tag (IMAGE_TAG)
    → Triggers THIS pipeline

CD Pipeline (this repo)
    → Receives IMAGE_TAG as a parameter
    → Updates deployment.yaml with the new image tag
    → Commits & pushes the change back to GitHub

ArgoCD (watching this repo)
    → Detects the change in deployment.yaml
    → Syncs and deploys the updated app to Kubernetes
```

This is the **GitOps pattern** — Git is the single source of truth for what runs in production.

---

## 🗂️ Repository Structure

```
e-to-e-deploy/
├── Jenkinsfile           # CD pipeline — updates manifest and pushes to Git
├── jenkinsCI             # CI pipeline reference script
├── deployment.yaml       # Kubernetes Deployment manifest
├── service.yml           # Kubernetes Service manifest
├── app-ingress.yml       # Ingress config for the application
└── argocd-ingress.yaml   # Ingress config for ArgoCD itself
```

---

## 🔧 Tech Stack

| Tool        | Role                                          |
|-------------|-----------------------------------------------|
| Jenkins     | CD automation — updates the manifest          |
| Kubernetes  | Container orchestration platform              |
| ArgoCD      | GitOps operator — watches Git and deploys     |
| Docker Hub  | Container image registry                      |
| NGINX Ingress | HTTP routing into the cluster              |
| GitHub SSH  | Secure Git operations from Jenkins            |

---

## 🛠️ CD Pipeline — What Each Stage Does

The `Jenkinsfile` runs on a labeled agent (`Node1`) and accepts an `IMAGE_TAG` parameter:

| Stage                         | What Happens                                                |
|-------------------------------|-------------------------------------------------------------|
| `Cleanup Workspace`           | Wipes the Jenkins workspace for a clean start               |
| `Checkout from SCM (SSH)`     | Clones this repo using SSH credentials                      |
| `Verify Files`                | Prints the current `deployment.yaml` so you can audit it    |
| `Update Deployment Manifest`  | Uses `sed` to replace the image tag in `deployment.yaml`    |
| `Commit & Push Changes (SSH)` | Commits the updated file and pushes it back to `main`       |

After the push, **ArgoCD automatically detects the change** and rolls out the new version to Kubernetes.

---

## ⚙️ Configuration

### Jenkins Credentials Required

| Credential ID   | Type       | Purpose                                        |
|-----------------|------------|------------------------------------------------|
| `github-ssh`    | SSH Key    | Allows Jenkins to clone and push to GitHub     |

### Pipeline Parameter

| Parameter   | Default  | Description                              |
|-------------|----------|------------------------------------------|
| `IMAGE_TAG` | `latest` | Docker image tag passed in from CI build |

---

## 🌐 Kubernetes Resources

### Deployment (`deployment.yaml`)
Defines how many replicas to run and which Docker image to use. The image line gets updated by the pipeline on every release.

### Service (`service.yml`)
Exposes the application inside the cluster.

### Ingress (`app-ingress.yml`)
Routes external HTTP traffic to the application service using NGINX Ingress Controller.

### ArgoCD Ingress (`argocd-ingress.yaml`)
Exposes the ArgoCD UI via Ingress so you can monitor deployments from a browser.

---

## 🚀 Setting This Up

1. **Set up ArgoCD** pointing to this repository (watching the `main` branch)
2. **Configure your CI pipeline** to call this pipeline's Jenkins job with the `IMAGE_TAG` parameter after a successful image push
3. **Add SSH credentials** in Jenkins with ID `github-ssh`
4. **Deploy** your Kubernetes resources:
   ```bash
   kubectl apply -f deployment.yaml
   kubectl apply -f service.yml
   kubectl apply -f app-ingress.yml
   ```
5. Every future code change will now flow automatically from commit → Docker image → deployed to Kubernetes

---

## 👤 Author

**Sudhakar** — [GitHub Profile](https://github.com/Sudhakar20000)
