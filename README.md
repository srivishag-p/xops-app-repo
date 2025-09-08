# XOps Task 6 – App Repository

A simple **NGINX-hosted static website** deployed via **GitOps** using **GitHub Actions** and **Argo CD** into a local **kind Kubernetes cluster**.

---

## Overview

This repository contains the application code for **XOps Microchallenge #6**.

* **index.html** – A styled static HTML page.
* **Dockerfile** – Builds and serves the site using NGINX.
* **.github/workflows/cicd.yaml** – CI/CD pipeline that:

  1. Builds the Docker image.
  2. Pushes it to Docker Hub (`<DOCKERHUB_USERNAME>/xops-app:<commit-sha>`).
  3. Argo CD to detect and sync the manifest changes into your local kind cluster.

---

## Quickstart (Local Workflow)

1. **Edit the UI**
   Modify `index.html` to update content or styling.

2. **Push changes**

   ```bash
   git add index.html
   git commit -m "Update UI styling"
   git push origin main
   ```

3. **Pipeline runs automatically**

   * GitHub Actions builds and pushes a new Docker image.
   * The CI workflow updates `deployment.yaml` in the Infra repo.
   * Argo CD auto-syncs the changes into Kubernetes.

4. **View the updated site**
   Forward the service to a local port:

   ```bash
   kubectl port-forward svc/xops-service 9090:80 -n default
   ```

   Open [http://localhost:9090](http://localhost:9090) in your browser.

---

## Project Structure

```
├── index.html
├── Dockerfile
├── .github/
│   └── workflows/
│       └── cicd.yaml
└── README.md
```

* **Dockerfile** – Based on `nginx:alpine`, serves `index.html`.
* **CI Workflow**:

  * Logs into Docker Hub.
  * Builds the image tagged with the Git commit SHA.
  * Pushes the image to Docker Hub.
  * Updates the Infra repo manifest.

---

## Required Setup

### On GitHub (App Repository)

Add the following **GitHub Secrets** under:
**Settings → Secrets and variables → Actions**

| Name                 | Value                                                              |
| -------------------- | ------------------------------------------------------------------ |
| `DOCKERHUB_USERNAME` | Your Docker Hub username                                           |
| `DOCKERHUB_TOKEN`    | Docker Hub personal access token                                   |
| `TOKEN`          | GitHub personal access token with repo/write access for Infra repo |

### Infra Repository (**xops-infra**)

Directory structure:

```
k8s/
├── deployment.yaml    # References image tag (updated by CI)
└── service.yaml       # Exposes the app via NodePort
```

Argo CD continuously watches this repo and deploys changes.

---

## Project Flow Summary

| Step            | Description                                        |
| --------------- | -------------------------------------------------- |
| 1. Edit Code    | Modify `index.html` as needed.                     |
| 2. Push Code    | `git push` triggers the CI pipeline.               |
| 3. CI Pipeline  | Builds & pushes Docker image + updates Infra repo. |
| 4. Infra Update | New image tag is committed to Infra repo.          |
| 5. Argo CD Sync | Detects changes and applies manifests to cluster.  |
| 6. Access App   | Port-forward service to view the updated site.     |

---

## Troubleshooting

* **Site not loading**: Ensure `kubectl port-forward` is running and the chosen port (e.g., 9090) is available.
* **Pipeline fails**: Double-check GitHub Secrets (especially `TOKEN` for Infra repo access).
* **App not updated**: Verify that `deployment.yaml` in Infra repo reflects the new image tag.
* **Argo CD OutOfSync**: Refresh or manually re-sync using the Argo CD UI or CLI.

