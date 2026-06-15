# Integration Test Scenarios

This directory contains Kustomize-based integration test scenarios for the Ceph project.

## Structure

```
integrations/
├── base/                           # Base integration test template
│   ├── integration-test-scenario.yaml
│   └── kustomization.yaml
├── overlays/                       # Version-specific overlays
│   ├── qe-build-email-7-1/
│   ├── qe-build-email-8-1/
│   ├── qe-build-email-9-0/
│   ├── qe-build-email-9-1/
│   ├── qe-build-email-9-2/
│   └── qe-jenkins-integration-9-2/
├── kustomization.yaml              # Top-level: combines all overlays
└── README.md                       # This file
```

## Base Template

The `base/` directory contains the common IntegrationTestScenario template with default values:
- Default Jenkins host URL
- Default job name
- Default pipeline reference
- Common labels and contexts

## Overlays

Each overlay directory customizes the base template for a specific Ceph version:

### qe-build-email overlays (7-1, 8-1, 9-0, 9.1, 9.2)
These use the standard Jenkins integration pipeline and only override:
- Integration test name
- Application name (ceph version)

### qe-jenkins-integration-9-2
This overlay uses a different Jenkins configuration and overrides:
- Integration test name
- Application name
- Jenkins host URL (https://149.81.216.83/)
- Job name (ceph-build-konflux-listener)
- Pipeline path (pipelines/qe_jenkins_integration.yaml)

## Usage

### Apply ALL integration tests at once

The top-level `kustomization.yaml` allows you to deploy all integration tests with a single command:

```bash
# Apply all integration tests
kubectl apply -k integrations/

# Or preview first
kustomize build integrations/
```

### Apply a specific integration test

```bash
# Apply qe-build-email-7-1
kubectl apply -k integrations/overlays/qe-build-email-7-1/

# Preview before applying
kustomize build integrations/overlays/qe-build-email-7-1/
```

### Generate manifests to files

```bash
# Generate all integration tests to a single file
kustomize build integrations/ > all-integration-tests.yaml

# Generate individual integration test
kustomize build integrations/overlays/qe-build-email-7-1/ > qe-build-email-7-1.yaml
```

## Adding a New Integration Test

1. Create a new overlay directory:
   ```bash
   mkdir -p integrations/overlays/qe-build-email-X-Y
   ```

2. Create a `kustomization.yaml` file with the necessary patches:
   ```yaml
   apiVersion: kustomize.config.k8s.io/v1beta1
   kind: Kustomization
   
   namespace: ceph-tenant
   
   resources:
   - ../../base
   
   patches:
   - patch: |-
       - op: replace
         path: /metadata/name
         value: qe-build-email-X-Y
       - op: replace
         path: /spec/application
         value: ceph-X-Y
     target:
       kind: IntegrationTestScenario
   ```

3. Test the build:
   ```bash
   kustomize build integrations/overlays/qe-build-email-X-Y
   ```

## Benefits of This Structure

1. **DRY Principle**: Common configuration is defined once in the base
2. **Easy Maintenance**: Changes to common fields only need to be made in one place
3. **Clear Separation**: Each version's unique configuration is isolated in its overlay
4. **Scalability**: Easy to add new versions or integration types
5. **Flexibility**: Can generate individual or all integration tests as needed
6. **Version Control**: Clear diff when changes are made to specific versions