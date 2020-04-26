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

TODO

## Deepdive videos

<iframe width="560" height="315" src="https://www.youtube.com/embed/videoseries?list=PLjJkiSWPwuPKG_3X7WsOtfr4MpcdjRzn7" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>



