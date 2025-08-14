# Project 1: GitOps based self service Multi-tenant Kubernetes Cluster

## Getting Started

1. You will need a running K8s cluster. I was using KinD in Docker Desktop for this. 
2. You will need to install [Helm](https://helm.sh/)
3. Once your K8s cluster is up and running you will need to install the following in the cluster:
   4. [ArgoCD](https://argo-cd.readthedocs.io/en/stable/getting_started/)
   5. [Crossplane](https://docs.crossplane.io/v2.0/get-started/)
   6. Crossplane Helm Provider: `kubectl apply -f crossplane-providers.yaml`
7. 