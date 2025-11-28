# Install ArgoCD
Refer to [Official ArgoCD documentation](https://argo-cd.readthedocs.io/en/stable/getting_started/), here the step to install ArgoCD :
```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```
Then i'm following this [tutorial](https://devopscube.com/setup-argo-cd-using-helm/#deploying-applications-from-github-with-argocd-gitops-way), to start deploying application with GitOps Way.