# Master Template

On this page you learn what is a Gimlet Master Template and how to tailor it to your needs.

!!! note ""

    This is an advanced guide, and unless you want to provide new possibilities for your developers, you can skip this one.

## Master Template

All applications deployed with Gimlet derive from the Master Template.

The Master Template is a generalized Kustomize template that is resolved to Kubernetes manifests at deploy time,
and then stored in the gitops repository for the gitops controller to deploy.

The Master Template provides the deployment options for developers to configure their deployments.
All [Deployment Options](http://localhost:8000/reference/options/) are made possible by the Master Template, and this is what developers can configure on the repository configuration UI, or in the `gimlet.yaml` file.

The Master Template is part of the GitOps repository, and you are free to look at it, or alter it to your needs.

!!! note ""

    See [The GitOps repository](/reference/gitops-repo) guide to get to know the GitOps repo.

## Structure

The Gimlet Master Template is a *Kustomize* based template.

Gimlet combines the values from the`.gimlet.yaml` file - or the deployment settings given on the UI - with the master template and constructs a valid application manifest and places in the `releases` folder.
Which is then synced to the cluster by Flux.

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

All deployment options are described in the `questions.yaml` file.
You are free to look at it, and change it to your team's needs.

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

### Variable types

#### String variables

String variables are made available in the Master Template yamls.

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

The Master Template yamls are Go Templates, you can inject any value from the `.gimlet.yaml` into a Master Template yaml.

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

#### Int variables

Int variables are made available in the Master Template yamls.

questions.yaml
```yaml
questions:
  - variable: replicas
    label: Replicas
    description: The number of instances to run
    type: int
    required: true
    default: 1
```

.gimlet.yaml
```yaml
envs:
  - env: staging
    name: demo-app
    replicas: 2
```

The Master Template yamls are Go Templates, you can inject any value from the `.gimlet.yaml` into a Master Template yaml.

kustomization.yaml
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
replicas:
  - name: {{ .name }}
    count: {{ .replicas }}
```

#### Int variables with a range of values

Int variables can also provide a range of valid values. If the `range` field is set, Gimlet will show a range picker widget on the UI.

```yaml
questions:
  - variable: replicas
    label: Replicas
    description: The number of instances to run
    type: int
    group: misc
    required: true
    default: 1
    range:
      min: 1
      max: 12
      step: 1
```

![Int ranges](./replicas.png)

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

#### Variables with Kustomize resources

Variables that define the `resource` field inject the referenced resource file to the Kustomize template.

questions.yaml
```diff
questions:
  - variable: ingress
    label: Ingress
    description: Exposes the application on a public URL, behind a firewall
    required: false
    default: null
    type: complex
    options: []
    group: networking
    patch: ""
+   resource: ingress.yaml
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
    name: myApp
    ingress:
      subdomain: test-app
```

The generated kustomization.yaml
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
bases:
  - ../../../config/myApp/
resources:
  - ingress.yaml
```

Using the `resource` field, you can add optional Kubernetes objects to your template: ingresses, volumes, etc.

#### Variables with Kustomize patches

Variables can also include a reference to a file in the `patch` field. If you set these variables for an application in 
the `.gimlet.yaml`, Gimlet will inject the referenced patch in the application's `kustomization.yaml`

With this, you can add optional fields to Kubernetes objects: resource requests, annotations, etc.

questions.yaml
```diff
  - variable: requests
    label: Resource Requests
    description:
    type: complex
    required: false
    group: resources
+   patch: resourceRequests.yaml
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

The generated kustomization.yaml
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
bases:
  - ../../../config/myApp/
patchesStrategicMerge:
  - resourceRequests.yaml
```

### Validation

#### Required fields

questions.yaml
```diff
questions:
  - variable: name
    label: Name
    description: The name of your application
    type: string
    group: basic
+   required: true
```

#### Minimum ratio between values

```diff
questions
  - variable: requests
    label: Resource Requests
    description: "Requests are used by the cluster scheduler to place your workload on the best suiting server node."
    type: complex
    required: false
    group: resources
    patch: resourceRequests.yaml
    subquestions:
      - variable: cpu
        label: CPU
        description: "Indicates the average CPU usage of the application. 1000 CPU shares = 1 CPU core"
        type: int
        default: 500
        range:
          min: 100
          max: 8000
          step: 100
        unit: "m"
        readableUnit: "shares"
+       validation:
+         - type: minRatio
+           variable: limits.cpu
+           value: 0.1
+           description: "CPU limits should be a maximum of 10X of CPU requests. We are over-provisioning our cluster on purpose, but anything over 10X could hurt uptime. Please adjust your request or limit."
```

Fails the validation if `value/targetVariableValue < rule.value`.

#### Maximum ratio between values

```diff
questions
  - variable: requests
    label: Resource Requests
    description: "Requests are used by the cluster scheduler to place your workload on the best suiting server node."
    type: complex
    required: false
    group: resources
    patch: resourceRequests.yaml
    subquestions:
      - variable: cpu
        label: CPU
        description: "Indicates the average CPU usage of the application. 1000 CPU shares = 1 CPU core"
        type: int
        default: 500
        range:
          min: 100
          max: 8000
          step: 100
        unit: "m"
        readableUnit: "shares"
+       validation:
+         - type: maxRatio
+           variable: limits.cpu
+           value: 1
+           description: "CPU requests can not be higher than CPU limits. Please adjust your request or limit."
```

Fails the validation if `value/targetVariableValue > rule.value`.

## Changing or extending the Master Template

To change the Master Template, you can edit any file in the template. Next time you deploy, Gimlet will update your application's manifest according to the new Master Template.

To extend the template, you have to add new variables in the `questions.yaml` file and change the existing template yamls, or add new Kustomize patch or resource files.

<!-- Fathom - beautiful, simple website analytics -->
<script src="https://cdn.usefathom.com/script.js" site="KVEHKPCQ" defer></script>
<!-- / Fathom -->
