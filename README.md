# Learn Platform Engineering

## Overview

This is my own little experimental repo for playing with the tools used in modern Platform Engineering. 
 
These include but are not limited to:
- Kubernetes
- ArgoCD
- Crossplane
- vCluster
- DAPR
- Backstage
- Kratix IO

## Project 1: GitOps based self service Multi-tenant Kubernetes Cluster

The idea: Extend K8s with a crossplane composite resource (XR) "Environment" that can be provisioned using a simple manifest. This resource will be a vCluster under the hood. This project will also use ArgoCD to implement a GitOps operational approach.



