# FAQ

??? note "Is Gimlet another CI engine?"

    No.
    
    Gimlet hooks into your existing CI setup, watches when your CI is finished, then kicks right in to deploy.
    
    Gimlet encapsulates all the deployment logic, so you can simplify your CI pipelines, and have a unified with more advanced deployment behavior, with good tooling.
    
    See more in [Deployment workflows](/developers/deploy-workflows)
    
??? note "Do I have to change anything in my CI pipeline?"
    
    No.
    
    Only deleting the deployment logic, since you won't need that anymore. I hope you like deleting code ;)
    
??? note "Can I run Gimlet and CI deployments in parallel?"

    Yes.
    
    Although I only recommend it while you are gaining confidence with Gimlet. Then you won't need CI to deploy anymore.
    
??? note "Is Gimlet a replacement of Helm?"
    
    No.
    
    They serve different purposes. In fact Gimlet wraps Helm (or Kustomize - up to you) to handle templating.
    
    Helm has many more features besides templating. In that sense, Gimlet **is** replacing Helm, you won't need those features from Helm.
    But Gimlet can't deny the ubiquity of Helm templates and the Helm templating knowledge in the ecosystem.
    
??? note "How does Gimlet compare to JenkinsX?"

    They are competing products. However Gimlet has a narrower focus, and aims to be less disruptive to adopt than JenkinsX.
    
??? note "How does Gimlet compare to FluxCD and ArgoCD?"

    Gimlet wraps FluxCD, see how on the [Architecture](/concepts/architecture) page.
    
    FluxCD is a brilliant tool, but you need glue to manage your environments, templating, etc. Gimlet is here to solve that.
    
    ArgoCD is a more feature rich GitOps tool. Gimlet and ArgoCD are more of a direct competitors.
    However Gimlet aims to assist not just in deployment, but in other areas as well, like application operation and developer on-boarding, education.