# The `.gimlet.yaml` file

In this guide you are going to deploy a new service on Kubernetes by creating the `.gimlet.yaml` file in your source code repository.

## `.gimlet.yaml`

The `.gimlet.yaml` file allows you to deploy or reconfigure an application in your source code editor.

In the [Deploying a new service](developers/deploy.md) you already saw how you can deploy an application through the Gimlet UI.
If you know the various options available to fine tune a deployment, you can use the `.gimlet.yaml` file to setup deployment more effectively.

Should you have an other application deployed with Gimlet already. Copying that file and use that as the base for the deployment, can be an effective approach.

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
  - env: staging
```

!!! note ""

    You can look up the available environments on the Gimlet UI, in the Settings page.

## Set the mandatory fields

Gimlet abstracts the many knobs and levers that a Kubernetes cluster exposes to developers, so you can start deploying quickly.

Set the few required fields to get started, Gimlet provides good defaults for the rest.

- **Strategy** - the deployment strategy to follow. Use *simple* for a rolling deployment.
- **Trigger** - a regular expression that matches the Github status line's name. Gimlet will trigger once this status line is set with success state.
- **Name** - the name of the deployment. Most likely the git repository name.
- **Namespace** - your Kubernetes administrator has already communicated this. Most likely it's going to be your team's namespace.
- **Image name** - the image name you set in your CI pipeline
- **Image tag** - the image tag that you set in your CI pipeline. You have the possibility to use a set of built in variables here. 

```yaml
envs:
  - env: staging
    strategy: simple
    trigger: .*/push
    name: website
    namespace: marketing
    image: gimlet.io/website
    tag: '${COMMIT_SHA:0:8}'
```

!!! note ""

    See the all the available deployment options in the [Deployment options](reference/options.md) reference.

## Deploy a revision

Push your changes to git. Once your CI pipeline is done, Gimlet will deploy your application.

Follow along the deployment on the Gimlet UI, or watch for Slack notifications.
