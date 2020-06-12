# The `.gimlet.yaml` file

In this guide you are going to deploy a new service on Kubernetes by creating the `.gimlet.yaml` file in your source code repository.

## `.gimlet.yaml`

The `.gimlet.yaml` file allows you to deploy or reconfigure an application in your source code editor.

In the [Deploying a new service](developers/deploy.md) you already saw how you can deploy an application through the Gimlet UI.
If you know the various options available to fine tune a deployment, you can use the `.gimlet.yaml` file to setup deployment more effectively.

Should you have another application deployed with Gimlet already. Copying that file and use as the base for the deployment can be an effective approach.

!!! note ""

    The UI and the `.gimlet.yaml` don't differ in feature set.

    But if you place a `.gimlet.yaml` in the root of your repository,
    you won't be able to change deployment settings on the UI anymore.

    This is reversible though, just delete the file, and you can manage your app on the UI again.
    Even the last settings of `.gimlet.yaml` will carry on in the UI.


## Create `.gimlet.yaml`

Let's start with creating a `.gimlet.yaml` in the root of your repository.

## Pick a deploy environment

You can manage all the deployments of your app in a single file.

```yaml
envs:
  staging:
```

!!! note ""

    You can look up the available environments on the Gimlet UI, in the Settings page.

## Set the mandatory fields

Gimlet abstracts the many knobs and levers that a Kubernetes cluster exposes to developers, so you can start deploying quickly.

Set the few required fields to get started, Gimlet provides good defaults for the rest.

- **Name** - the name of the deployment. Most likely the git repository name.
- **Namespace** - your Kubernetes administrator has already communicated this. Most likely it's going to be your team's namespace.
- **Image name** - the image name you set in your CI pipeline
- **Image tag** - the image tag that you set in your CI pipeline. You have the possibility to use a set of built in variables here. 

```yaml
envs:
  staging:
    - name: website
      namespace: marketing
      image: gimlet.io/website
      tag: '${COMMIT_SHA:0:8}'
```

!!! note ""

    See the all the available deployment options in the [Deployment options](reference/options.md) reference.

## Push and deploy

Push your changes now, your repository should show up in Gimlet as configured.

Make an ad-hoc deploy now. See [Deploy a revision](/developers/deploy#deploy-a-revision) section from the Deploying a new service guide to see how to deploy from the UI.

This first manual deployment is necessary for Gimlet to register webhooks on your repository, and it's also a good exercise to see the deployment succeed. 

## Set an automatic deployment policy

Extend your `.gimlet.yaml` with a deployment policy to enable automatic deployments.

The following example shows a simple CD setup to staging.
Check the [Deployment policy examples](/developers/deploy-workflows#deployment-policy-examples) to find the policy that suits your needs.

```diff
envs:
  staging:
    - name: website
      namespace: marketing
      image: gimlet.io/website
      tag: '${COMMIT_SHA:0:8}'
+      deploy:
+        event: push
+        branch: master
+        waitForCI: true
```

Once you are done, push your changes and verify the behavior.

<!-- Fathom - beautiful, simple website analytics -->
<script src="https://cdn.usefathom.com/script.js" site="KVEHKPCQ" defer></script>
<!-- / Fathom -->
