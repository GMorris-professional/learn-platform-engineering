# Crossplane Composition Guide - vcluster Development Environment

## Overview
This Crossplane Composition creates isolated development environments using vcluster (virtual Kubernetes clusters). It demonstrates how to use patches to make compositions dynamic and reusable.

**Composition Name:** `dev.env.bigly.co.za`  
**Composite Type:** `XEnvironment` (xbiglylabs.co.za/v1alpha1)  
**Purpose:** Creates isolated dev environments with virtual Kubernetes clusters

## Architecture

The composition creates two main resources:
1. **vcluster Helm Release** - The virtual Kubernetes cluster
2. **Helm ProviderConfig** - Enables Helm operations within the vcluster

## Key Components

### Base Configuration
- **Chart:** vcluster v0.15.7 from https://charts.loft.sh
- **Features:** 
  - Multi-namespace mode enabled
  - Fallback host DNS resolution
  - Configurable syncer arguments

### Patches - Dynamic Configuration Engine

Patches take values from the composite resource (XEnvironment) and dynamically populate managed resource fields:

#### vcluster Helm Release Patches

1. **Namespace Assignment**
   ```yaml
   fromFieldPath: metadata.name → spec.forProvider.namespace
   ```
   - Deploys vcluster to namespace matching XEnvironment name

2. **External Name Tracking**
   ```yaml
   fromFieldPath: metadata.name → metadata.annotations[crossplane.io/external-name]
   ```
   - Sets Crossplane tracking annotation

3. **Resource Naming with Transform**
   ```yaml
   fromFieldPath: metadata.name → metadata.name
   transform: "%s-vcluster"
   ```
   - Appends "-vcluster" suffix to resource name

4. **Kubeconfig Secret Configuration**
   ```yaml
   CombineFromComposite: "--out-kube-config-secret=%s-secret"
   ```
   - Configures where vcluster stores its kubeconfig

5. **API Server URL Configuration**
   ```yaml
   CombineFromComposite: "--out-kube-config-server=https://%s.%s.svc"
   ```
   - Sets internal Kubernetes service URL for API access

6. **TLS Certificate SAN**
   ```yaml
   CombineFromComposite: "--tls-san=%s.%s.svc"
   ```
   - Ensures SSL certificates work with service DNS names

#### Helm ProviderConfig Patches

1. **Secret Name with Prefix**
   ```yaml
   fromFieldPath: metadata.name → spec.credentials.secretRef.name
   transform: "vc-%s"
   ```

2. **Secret Namespace**
   ```yaml
   fromFieldPath: metadata.name → spec.credentials.secretRef.namespace
   ```

3. **ProviderConfig Name**
   ```yaml
   fromFieldPath: metadata.name → metadata.name
   ```

## Example Usage

When you create an XEnvironment named `team-alpha`, the composition generates:

**vcluster Deployment:**
- Namespace: `team-alpha`
- Resource name: `team-alpha-vcluster`
- Syncer arguments:
  - `--out-kube-config-secret=team-alpha-secret`
  - `--out-kube-config-server=https://team-alpha.team-alpha.svc`
  - `--tls-san=team-alpha.team-alpha.svc`

**ProviderConfig:**
- Name: `team-alpha`
- Secret reference: `vc-team-alpha` in `team-alpha` namespace

## Readiness Checks

- **vcluster**: Waits for Helm release state = "deployed"
- **ProviderConfig**: No readiness check (type: None)

## Key Benefits

1. **Reusability**: One composition template creates multiple isolated environments
2. **Dynamic Configuration**: Patches eliminate hardcoded values
3. **Proper Isolation**: Each environment gets its own namespace and vcluster
4. **Extensibility**: ProviderConfig enables further Helm deployments within vclusters

## Patch Types Used

- **Simple Field Path**: Direct value copying
- **Transform**: Modify values with string formatting
- **CombineFromComposite**: Merge multiple fields with custom formatting

## Connection Secrets
- Stored in: `crossplane-system` namespace
- Used for: Credentials and kubeconfig management