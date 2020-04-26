# Deployment workflows

So far you have seen how to deploy any git revision on-demand.

On this page you will learn how Gimlet can be added to your software delivery pipeline and automate deployment on common events like git push to master, or git tag.

## How you used to deploy with CI

In a typical CI pipeline, you latch on the deployment step as the very last step of the workflow, 
and you control when to deploy your application to a given environment with branch and event conditions.

You develop a script or use a CI plugin for the deploy, and parameterize each deploy step with environment specific variables.

With multiple environments, rollback steps this becomes hairy, and while it scales to some degree, 
pipeline configuration diverge from project to project and becomes a maintenance burden.

## Gimlet hooks into your existing CI pipeline through Github statuses

Gimlet encapsulates all the deployment logic, so your CI pipeline can focus on things it really shines at. Running test, generating build artifacts.

Gimlet listens on Github Status lines and kicks in when your CI is done, and reports success back to Github through a status line.

You have seen Github Status lines before on your Pull Requests, showing if your automated tests are passing.
Or by clicking on the green tick mark in your commit list on Github.

## Gimlet Triggers

A Gimlet Trigger is a regular expression on Github Status line context.

Gimlet runs if a status is published to Github with a success state, and the context matches the regular expression.

#### Github status examples
Don't worry, you don't have to publish anything from your CI pipeline.

CI engines send Github Statuses automatically: CircleCI sends a status line for each job in your workflow, others like Drone.io send one per triggered job.

- `ci/circleci: e2e-test` for example is sent when your job - preassumably running end-to-end tests - is done

- `continuous-integration/drone/push` indicates a job triggered by a git push

#### Gimlet Trigger examples

Given a CircleCI status line `ci/circleci: docker-build-and-push`, 
a Gimlet trigger that runs every time there is a new image can be written as `.*docker-build-and-push`


## Deploying on common workflow events with Gimlet

#### Deploying any version on-demand

On-demand deploy is done on the UI. See [Deploy a revision section](/developers/deploy#deploy-a-revision) from the Deploying a new service guide.

#### Deploy automatically when a new image is built

Given a CircleCI job where you build and push a docker image, with name `docker-build-and-push`. CircleCI publishes a status line `ci/circleci: docker-build-and-push`, 

The Gimlet trigger that runs every time there is a new image is written as `.*docker-build-and-push`

#### Deploy on git push

Given a Drone.io pipeline that builds and publishes a Docker image on git push to the master branch.

Drone.io sends a `continuous-integration/drone/push` status line to Github, and you can write `.*/push` as a Gimlet trigger.

!!! note ""

    Gimlet only regards status lines on the master branch at this point. 

#### Deploy on git tag

Given a Drone.io pipeline that builds and publishes a Docker image on git tag on the master branch.

Drone.io sends a `continuous-integration/drone/tag` status line to Github, and you can write `.*/tag` as a Gimlet trigger.

!!! note ""

    Gimlet only regards status lines on the master branch at this point. 
