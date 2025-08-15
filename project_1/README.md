# Project 1: GitOps based self service Multi-tenant Kubernetes Cluster

## Getting Started

* You will need a running K8s cluster. I was using KinD in Docker Desktop for this. 
* You will need to install [Helm](https://helm.sh/)
* Once your K8s cluster is up and running you will need to install the following in the cluster:
   * [ArgoCD](https://argo-cd.readthedocs.io/en/stable/getting_started/)
   * [Crossplane](https://docs.crossplane.io/v2.0/get-started/)
   * Crossplane Helm Provider: `kubectl apply -f crossplane-providers.yaml`
* Expose the ArgoCD UI: `kubectl port-forward ` and login:
  * username: `admin`
  * password obtained from: `kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo`
* Add the Github Repo as a new app in ArgoCD
* Add the Environment Resource XRD and XR to cluster:
  * `kubectl apply -f environment-xrd.yaml`
  * `kubectl apply -f environment-composition.yaml`

## Diving Deeper

* [Environment XRD Explanation](docs/XRD_EXPLANATION.md)
* [Environment Composition Explanation](docs/COMPOSITION_EXPLANATION.md)