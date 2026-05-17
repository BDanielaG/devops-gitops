# End-to-end GitOps Deployment

This project demonstrates a GitOps workflow using Kubernetes and Argo CD.
The application is deployed automatically from a GitHub repository into a Kubernetes cluster running locally with Minikube. Argo CD continuously monitors the repository and synchronizes the state of the cluster with the configuration stored in github.
The project includes:

-Kubernetes

-Minikube

-Argo CD

-Docker

-GitHub

-kubectl


All Kubernetes manifests and deployment configurations are stored in GitHub. Argo CD monitors the repository and automatically applies changes to the cluster whenever updates are pushed to the main branch.
# Kubernetes Deployment

The application is deployed using Kubernetes yaml files.

The deployment configuration defines:

-the application container

-replica count

-networking configuration

-Kubernetes service exposure

The application runs with 2 replicas inside the cluster.
Argo CD was configured with:

-automatic synchronization

-self-healing enabled

-pruning enabled


```
syncPolicy:
  automated:
    prune: true
    selfHeal: true
```

This allows the cluster to remain synchronized with the Git repository automatically.

---

# Running the Project

First we have to start the minikube

```
minikube start
```

---

Also install argo CD

```
kubectl create namespace argocd
```

```
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

---
Access the Argo CD UI

```
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Open:

```
https://localhost:8080
```

Get the initial password:

```
kubectl -n argocd get secret argocd-initial-admin-secret \
-o jsonpath="{.data.password}" | base64 -d
```

Username:

```
admin
```

---

Deploy the app

```
kubectl apply -f argocd/application.yaml
```

Argo CD automatically deploys the application from the GitHub repository.

---

Access the app

```
minikube service gitops-app-service
```

---

# GitOps Workflow

Any modification pushed to the GitHub repository is automatically detected by Argo CD and after synchronization kubernetes resources are updated automatically and the application state matches the git repository state.
No manual deployment steps are required after pushing changes to GitHub.

---


# To test the self healing mechanism, this were the steps:

The deployment was manually modified using:

```
kubectl scale deployment gitops-app --replicas=0
```

This created a difference between:

-the configuration stored in git

-the actual cluster state

Argo CD detected the differences and automatically restored the deployment back to 2 replicas because self-healing was enabled.

---

# Conclusion
Using Argo CD and Kubernetes allows infrastructure and application state to remain consistent with the configuration stored in Git.
