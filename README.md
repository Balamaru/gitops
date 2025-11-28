# 🚀 GitOps with Argocd and Gitlab (ce)
The purpose of this project is to build a gitops pipeline with Argocd and Gitlab. I will be using my own Gitlab server, register a gitlab-runner, configure the pipeline to build a Docker image, push it to the Docker hub, and deploy it to a server using SSH. For demonstration purpose, I will deploy a small, static web page using Nginx.

# 🔧 Prerequisite
- Kubernetes cluster
- 2 servers Ubuntu 22.04 (gitlab-server, gitlab-runner)
- (Optional, but recommended) Registered domain-name
- Docker hub account

## 🚀 GitOps Deployment Flow
This repository uses a GitOps approach to deploy applications using ArgoCD.

All changes you want to deploy to Kubernetes will be made through changes to the YAML files in the *argocd/* folder.
## 📦 Repository Structure
```bash
gitops
├── argocd/
│   ├── go-app.yaml
│   └── node-app.yaml
├── gitlab/
│   ├── maifest/
│   └── services/
│       ├── go-app/
│       └── nodejs-app/
├── .gitlab-ci-argo.yml
└── .gitlab-ci.yml
```
- *gitlab/services/* contains the source code of each application.
- *argocd/* contains GitOps manifest for ArgoCD
- *.gitlab-ci-argo.yml* contains pipeline build → push image → update manifest → ArgoCD sync.
- *.gitlab-ci.yml* contains pipeline build → push image → update manifest → copy manifest to kubernetes control-plane → apply manifest from cp.
## ⚙️ CI/CD Flow (GitOps Flow)
Here is the automated GitOps flow used:
### 1️⃣ Developer Push Code
Developer made a commit & push ke repository, example:
- script Go (*gitlab/services/go-app*)
- script NodeJS (*gitlab/services/nodejs-app*)
### 2️⃣ GitLab CI Build & Push Docker Image
GitLab CI would:
- Build Application Docker image
- Push image to container registry
- Generate tag based on commit SHA (*$CI_COMMIT_SHA*)

Example image output:
```sh
registry.example.com/username/go-app:e54cdd31846f09
registry.example.com/username/node-app:e54cdd31846f09
```
### 3️⃣ GitLab CI Update File GitOps (ArgoCD)
Job *update_manifests* would:
- Change tag image in file:
    - *argocd/go-app.yaml*
    - *argocd/node-app.yaml*

Similiar like this command:
```sh
sed -i "s|image: .*/go-app:.*|image: ${IMAGE_GO}:${TAG}|g" argocd/go-app.yaml
```
Then commit to branch main:
```sh
git commit -m "Update images to $CI_COMMIT_SHA [skip ci]"
```
**✔ Why should having [skip ci]?**

To prevent an infinite CI loop, since the manifest update commit will trigger a new pipeline. With *[skip ci]*, the commit will not run the pipeline.
### 4️⃣ GitLab CI Push Changes to Main Branch
CI will push the updated manifest file.:
```sh
git push origin HEAD:main
```
In order to push from CI, the repository uses:
- Deploy Token
- Or CI Job Token (if permitted)

Tokens must have permissions:
- *write_repository*
- *read_repository*
### 5️⃣ ArgoCD Pulls Changes Automatically
Because ArgoCD is configured with:
```yaml
syncPolicy:
  automated:
    prune: true
    selfHeal: true
```
So every time there is a new commit on the main branch:
- ArgoCD auto-sync
- Deployments within the cluster will be updated
- The latest image tag will be deployed
## 🔄 GitOps Flow Summary
```sh
Developer commit code
        ↓
GitLab CI build image
        ↓
GitLab CI push image to registry
        ↓
GitLab CI update manifest (argocd/*.yaml)
        ↓
GitLab CI commit + push
        ↓
ArgoCD detect changes
        ↓
ArgoCD deploys to a Kubernetes cluster
```
## 🛡️ Important Points
**✔ Avoiding Infinite Loop**

Commit update manifest using *[skip ci]* to avoid triggering a new pipeline.

**✔ Push Access from CI**

Make sure CI uses the Credentials it has (by using a personal access token):
- *write_repository*
- *read_repository*

For example using:
```sh
GITLAB_CI_USER=my-deploy-user
GITLAB_CI_TOKEN=my-deploy-token
```