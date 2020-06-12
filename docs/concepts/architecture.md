# Architecture

Gimlet has a client-server architecture.

## Overview

![Gimlet architecture](./architecture.svg)

**Gimlet Agent**

The agent runs inside your Kubernetes cluster. It gathers realtime information and ships it to the Gimlet server.
This information is used to power the realtime cluster view in the Gimlet UI and to determine error states in your cluster.

You run as many Gimlet Agents as many environments you have.

**Gimlet Server**

Runs either hosted on [https://gimlet.io](https://gimlet.io) or you self-host it in your infrastructure.

The Gimlet Server

- powers the Gimlet UI
- aggregates environment information received from the Gimlet Agents
- processes webhooks from git
- runs the GitOps event handlers and updates the GitOps Repository

**GitOps repository**

The GitOps Repository holds the Kubernetes manifests of your applications. 

**FluxCD**

FluxCD is the GitOps controller. It continuously applies manifest changes to the Kubernetes cluster.

## How components interact

When a Gimlet deployment is triggered - either on-demand from the UI, or by a workflow trigger - the Gimlet Server looks for the deployment configuration
and generates an updated application manifest, which it then puts in the GitOps repository.

FluxCD is deployed in the deployment environment and listens for changes in the GitOps repository's releasees folder.
If it sees a change, it will apply it on the cluster.

The Gimlet Agent is also running in the deployment environment, and streams a realtime Kubernetes state to the Gimlet Server. Which then streams to the UI.


!!! note ""

    To learn more about the GitOps repository, see the [The GitOps repository](/reference/gitops-repo) page
    
    To learn more about on-demand deploys and service configuration, see the [Deploying a new service](/developers/deploy) page

    To learn more about workflow triggers, see the [Deployment workflows](/developers/deploy-workflows) page



## Deepdive videos

<iframe width="560" height="315" src="https://www.youtube.com/embed/videoseries?list=PLjJkiSWPwuPKG_3X7WsOtfr4MpcdjRzn7" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

<!-- Fathom - beautiful, simple website analytics -->
<script src="https://cdn.usefathom.com/script.js" site="KVEHKPCQ" defer></script>
<!-- / Fathom -->

