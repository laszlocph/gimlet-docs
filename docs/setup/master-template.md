# Master Template

On this page you learn what is a Gimlet Master Template and how to tailor it to your needs.

!!! note ""

    This is an advanced guide, and unless you want to provide new possibilities for your developers, you can skip this one.

## Master Template

Gimlet uses a generic template for new deployments.

When you create a new deployment, or change an existing one, the deployment settings on the UI, and the fields available in `.gimlet.yaml` are coming from the Master Template.

The Master Template is part of the GitOps repository, and you are free to look at it, or alter it to your needs.

!!! note ""

    See [The GitOps repository](/reference/gitops-repo) guide to get to know the GitOps repo.

## Structure

The Gimlet Master Template is a *Kustomize* based template.

Gimlet combines the values from the`.gimlet.yaml` file, or the deployment settings given on the UI, with the master template and constructs and places a valid application manifest to the `releases` folder.
Which is then synced to the cluster by Flux.

The available options are defined in the `questions.yaml` which is then merged with the application settings to get the application yamls 

```bash
masterTemplate
├── base
│   ├── deployment.yaml
│   ├── kustomization.yaml
│   └── service.yaml
├── instance
│   ├── horizontal-pod-autoscaler.yaml
│   ├── kustomization.yaml
│   └── test-ingress.yaml
├── patches
│   ├── containerPort.yaml
│   ├── envVars.yaml
│   ├── image-pull-policy-patch.yaml
│   ├── prometheus-scrape-patch.yaml
│   ├── resourceLimits.yaml
│   └── resourceRequests.yaml
└── questions.yaml
``` 

## `questions.yaml`

The following sample of the `questions.yaml` shows a few types of variables.

```yaml
questions:
  - variable: name
    label: Name
    description: The name of your application
    type: string
    group: basic
    required: true
  - variable: tag
    label: Image tag
    description: The image tag to apply on release. Supports bash string manipulations
    type: string
    group: basic
    required: true
    default: "${COMMIT_SHA:0:8}"
  - variable: containerPort
    label: Container Port
    description: The port your application is listening on
    type: string
    group: misc
    required: false
    patch: containerPort.yaml
  - variable: securedTestIngress
    label: Secured Test Ingress
    description: Exposes the application publicly on a secured ingress on test
    type: complex
    required: false
    options: []
    group: networking
    resource: test-ingress.yaml
    subquestions:
      - variable: subdomain
        label: Subdomain
        description: Specifies the subdomain your app is going to be exposed on <<subdomain>>.test.raffle.systems
        type: string
  - variable: requests
    label: Resource Requests
    description:
    type: complex
    required: false
    group: resources
    patch: resourceRequests.yaml
    subquestions:
      - variable: cpu
        label: CPU
        description:
        type: string
      - variable: memory
        label: Memory
        description:
        type: string
```

#### String variables

String variables are made available in the master template yamls, and you can use them as Go Templates.

questions.yaml
```yaml
questions:
  - variable: name
    label: Name
    description: The name of your application
    type: string
    group: basic
    required: true
```

.gimlet.yaml
```yaml
envs:
  - env: staging
    name: demo-app
```

```yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .name }}
spec:
  template:
    spec:
      containers:
      - name: {{ .name }}
        image: dummy-image
```

#### Complex variables

Complex variables are also made available as Go Templates, they define a hierarchy.

questions.yaml
```yaml
  - variable: requests
    label: Resource Requests
    description:
    type: complex
    required: false
    group: resources
    patch: resourceRequests.yaml
    subquestions:
      - variable: cpu
        label: CPU
        description:
        type: string
      - variable: memory
        label: Memory
        description:
        type: string
```
.gimlet.yaml
```yaml
  - env: production
    limits:
      cpu: 100m
      memory: 128Mi
```

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .name }}
spec:
  template:
    spec:
      containers:
        - name: {{ .name }}
          resources:
            limits:
              memory: {{ .limits.memory }}
              cpu: {{ .limits.cpu }}
```

#### Variables with Kustomize resource

Variables can also include a reference to a file in the `resource` field. If you set these variables for an application in 
the `.gimlet.yaml`, Gimlet will inject the referenced resource in the application's `kustomization.yaml`

With this, you can add optional Kubernetes objects to your template: ingresses, volumes, etc.

questions.yaml
```yaml
- variable: ingress
  label: Ingress
  description: Exposes the application on a public URL, behind a firewall
  required: false
  default: null
  type: complex
  options: []
  group: networking
  patch: ""
  resource: ingress.yaml
  subquestions:
  - variable: subdomain
    label: Subdomain
    description: Specifies the subdomain your app is going to be exposed on <<subdomain>>.test.laszlo.cloud
    required: false
    default: null
    type: string
    options: []
    group: ""
    patch: ""
    resource: ""
    subquestions: []
```

.gimlet.yaml
```
  - env: production
    ingress:
      subdomain: test-app
```


#### Variables with Kustomize patch

Variables can also include a reference to a file in the `patch` field. If you set these variables for an application in 
the `.gimlet.yaml`, Gimlet will inject the referenced patch in the application's `kustomization.yaml`

With this, you can add optional fields to Kubernetes objects: resource requests, annotations, etc.

questions.yaml
```yaml
  - variable: requests
    label: Resource Requests
    description:
    type: complex
    required: false
    group: resources
    patch: resourceRequests.yaml
    subquestions:
      - variable: cpu
        label: CPU
        description:
        type: string
      - variable: memory
        label: Memory
        description:
        type: string
```
.gimlet.yaml
```yaml
  - env: production
    limits:
      cpu: 100m
      memory: 128Mi
```

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .name }}
spec:
  template:
    spec:
      containers:
        - name: {{ .name }}
          resources:
            limits:
              memory: {{ .limits.memory }}
              cpu: {{ .limits.cpu }}
```

## Changing or extending the Master Template

To change the Master Template, you can edit any file in the template. Next time you deploy, Gimlet will update your application's manifest according to the new Master Template.

To extend the template, you have to add new variables in the `questions.yaml` file and change the existing template yamls, or add new Kustomize patch or resource files.

<!-- Fathom - beautiful, simple website analytics -->
<script src="https://cdn.usefathom.com/script.js" site="KVEHKPCQ" defer></script>
<!-- / Fathom -->
