# The GitOps repository

On this page you will learn all there is to the GitOps repository.

## The GitOps repository

Gimlet follows the GitOps approach, and stores all application manifests in a single git repository.

The GitOps controller pulls the desired state from the GitOps repository, and applies it on the Kubernetes cluster.

## Architecture

Gimlet encapsulates all the logic to manage applications, environments in the GitOps repository, and uses FluxCD as the GitOps controller.

![Gimlet architecture](../concepts/architecture.svg)

See the [Architecture](/concepts/architecture) guide for a full description.

## GitOps repository structure

Gimlet uses the following folder structure in the GitOps repo:

!!! note ""

    It describes the structure when *Kustomize* is used for the templates.

```bash
.
├── masterTemplate
│   ├── base
│   │   ├── deployment.yaml
│   │   ├── kustomization.yaml
│   │   └── service.yaml
│   ├── instance
│   │   ├── horizontal-pod-autoscaler.yaml
│   │   ├── kustomization.yaml
│   │   └── ...
│   ├── patches
│   │   ├── containerPort.yaml
│   │   ├── envVars.yaml
│   │   ├── image-pull-policy-patch.yaml
│   │   ├── prometheus-scrape-patch.yaml
│   │   ├── ...
│   └── questions.yaml
└── releases
    ├── production
    │   └── app-1
    │   │   ├── service.yaml
    │   │   └── deployment.yaml
    └── staging
        ├── app-1
    │   │   ├── service.yaml
    │   │   └── deployment.yaml
        ├── demo-app
    │   │   ├── service.yaml
    │   │   └── deployment.yaml
```

- **masterTemplate** Stores a generic application blueprint. Gimlet uses this template for all new deployments.
- **releases** is where the deployed application configuration is. FluxCD listens on these folders.

!!! note ""

    See the [Master Template](/setup/master-template) guide to see how the Master Template is structured.

<!-- Fathom - beautiful, simple website analytics -->
<script src="https://cdn.usefathom.com/script.js" site="KVEHKPCQ" defer></script>
<!-- / Fathom -->
