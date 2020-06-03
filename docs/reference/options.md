# Deployment options

| Variable     | Description           | Type | Example |
| ------------- |-------------| -----| ----|
| name     | The name of your application | string | `account-service` |
| namespace      | The namespace to deploy your application in | string | `core-team` |
| image | The image to deploy, without the tag | string | `mycompany/account-service` |
| tag | The tag of the deployed image <br /><br /> Use the `COMMIT_SHA` and `TAG` variables to inject git metadata. Apply string functions to match your image tag pattern | string | `${COMMIT_SHA}` the full hash <br /> `${COMMIT_SHA:0:8}`  hash with its first 8 characters |
| containerPort | The port your application is listening on | string | `8080` |
| ingress.subdomain | The subdomain your app is going to be exposed on <<subdomain>>.your-ingress-domain.tld | string | `accounts` |
| requests.cpu | Your average CPU usage. This is what Kubernetes is using to place your workload. | string | `100m` - 1/10 of a CPU core |
| requests.memory | Your average memory usage. This is what Kubernetes is using to place your workload. | string | `128Mi` |
| limits.cpu | Your maximum CPU usage. It's a hard ceiling, you won't get more than this. | string | `2` - two full cores |
| limits.memory | Your maximum memory usage. It's a hard ceiling, your app will be restarted if you reach it | string | `256Mi` |
| vars | An array of environment variables for your app | map | `DATABASE_DB: mydb` |
| prometheusScraping | Enables Prometheus scraping by adding the prometheus.io/scrape annotation on the pod | bool | `true` |
| alwaysPullImages | If true, every deploy or restart will check the registry if it has a different image sha of the set image tag | bool | `true` |

!!! note ""
    
    This is an ever increasing list.
    
    
    If you would like to change or add new deployment options, check out the [Master Template](|setup|master-template) guide.