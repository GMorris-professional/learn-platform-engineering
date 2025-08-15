# Crossplane XRD Guide - XEnvironment Definition

## Overview
This Composite Resource Definition (XRD) creates the API schema for requesting development environments. It defines the user interface that the vcluster Composition implements.

**Resource Name:** `xenvironments.xbiglylabs.co.za`  
**API Group:** `xbiglylabs.co.za`  
**Version:** `v1alpha1`

## Dual Resource Architecture

This XRD defines two related Kubernetes resource types:

### 1. Composite Resource - XEnvironment
- **Kind:** `XEnvironment`
- **Plural:** `xenvironments`
- **Short Names:** `xenv`, `xenvs`
- **Scope:** Cluster-wide
- **Usage:** Platform teams, infrastructure management
- **Purpose:** The actual composite resource that triggers Compositions

### 2. Claim Resource - Environment  
- **Kind:** `Environment`
- **Plural:** `environments`
- **Short Names:** `env`, `envs`
- **Scope:** Namespace-scoped
- **Usage:** Developers, end users
- **Purpose:** User-friendly interface for requesting environments

## Custom kubectl Columns

When running `kubectl get xenvironments`, additional columns display:

| Column | Source | Purpose |
|--------|--------|---------|
| **CONNECT-TO** | `.spec.resourceRef.name` | Shows which vcluster to connect to |
| **TYPE** | `.spec.compositionSelector.matchLabels.type` | Environment type (e.g., "development") |

```bash
# Example output
kubectl get xenv
NAME              CONNECT-TO        TYPE           AGE
my-dev-xyz123     my-dev-vcluster   development    5m
```

## Schema Definition

**Current v1alpha1 Schema:**
```yaml
spec:
  test:
    type: boolean  # Optional field
```

**Schema Characteristics:**
- Minimal configuration required
- Relies primarily on `metadata.name` for resource naming
- The `test` field appears to be a simple boolean flag
- Schema suggests early development stage

## Workflow Example

### 1. Developer Creates Claim
```yaml
apiVersion: xbiglylabs.co.za/v1alpha1
kind: Environment
metadata:
  name: my-dev-env
  namespace: team-alpha
spec:
  test: true
```

### 2. Crossplane Creates Composite
```yaml
apiVersion: xbiglylabs.co.za/v1alpha1
kind: XEnvironment
metadata:
  name: my-dev-env-xyz123  # Auto-generated
spec:
  test: true
  # Crossplane adds additional fields
```

### 3. Composition Execution
The XEnvironment triggers the vcluster Composition which creates:
- vcluster Helm release named `my-dev-env-xyz123-vcluster`
- Namespace: `my-dev-env-xyz123`
- Service URL: `https://my-dev-env-xyz123.my-dev-env-xyz123.svc`
- ProviderConfig for internal Helm operations

## Relationship to Composition

**XRD defines the API:**
```yaml
# This XRD creates the XEnvironment type
group: xbiglylabs.co.za
names:
  kind: XEnvironment
```

**Composition implements the API:**
```yaml
# Composition references the XRD-defined type
compositeTypeRef:
  apiVersion: xbiglylabs.co.za/v1alpha1
  kind: XEnvironment
```

## Common Commands

```bash
# Create environment (as developer)
kubectl apply -f environment.yaml

# List environments in namespace
kubectl get env

# List all environments cluster-wide  
kubectl get xenv

# Get detailed info
kubectl describe env my-dev-env

# Check status and connection info
kubectl get xenv -o wide
```

## Schema Evolution Potential

Current schema is minimal. Production-ready versions might include:

```yaml
spec:
  # Resource specifications
  resources:
    cpu: "2"
    memory: "4Gi" 
    storage: "10Gi"
  
  # Kubernetes version
  version: "1.28"
  
  # Optional addons
  addons:
    - name: monitoring
      enabled: true
    - name: logging
      enabled: false
  
  # Environment lifecycle
  ttl: "7d"
  autoSuspend: true
  
  # Network configuration
  networking:
    ingress: true
    loadBalancer: false
```

## Key Benefits

1. **Abstraction**: Developers don't need to know about vclusters or Helm charts
2. **Self-Service**: Teams can create environments on-demand
3. **Consistency**: All environments follow the same template
4. **Governance**: Platform teams control the underlying implementation
5. **Discoverability**: Standard Kubernetes API patterns (`kubectl get env`)

## Version Management

- **Served**: `true` - API accepts requests for this version
- **Referenceable**: `true` - Other resources can reference this version
- **Storage**: Implied as storage version (only version defined)

## Integration Points

This XRD works with:
- **Compositions**: Define how to fulfill XEnvironment requests
- **ProviderConfigs**: Configure providers for resource creation  
- **RBAC**: Control who can create/manage environments
- **Admission Controllers**: Validate and mutate requests