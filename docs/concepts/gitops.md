# Gitops

Many GitOps interpretations circulate out there.

## Definition

Gimlet uses the term in the strict [Weave Works definition](https://www.weave.works/technologies/gitops/#what-is-gitops):

> "GitOps is a way to do Kubernetes cluster management and application delivery.  It works by using Git as a single source of truth for declarative infrastructure and applications. With Git at the center of your delivery pipelines, developers can make pull requests to accelerate and simplify application deployments and operations tasks to Kubernetes."

To pin it even more down, Gimlet is using the GitOps term as a synonym of the FluxCD and ArgoCD implementations.

## In a nutshell

CI pipelines do not apply (with kubectl or helm) Kubernetes manifests on the cluster anymore. Instead, you record the desired state in a git repository, 
and the GitOps controller is responsible to apply the changes on the Kubernetes cluster.

The GitOps controller can be looked at as an endless loop that runs `kubectl apply` if anything changes in git. Gimlet uses FluxCD as the GitOps controller.

Gimlet is encapsulating all the logic to update the gitops repository in a consistent, environment aware fashion.

The GitOps abstraction allows Gimlet to package much of the logic that previously was scattered in CI pipelines and implemented by every company.