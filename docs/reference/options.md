# Deployment options

| Variable     | Description           | Type | Example |
| ------------- |-------------| -----| ----|
| name     | The name of your application | string | |
| namespace      | The namespace to put your application in | string | |
| image | The image to deploy | string | |
| tag |       | string | |
| containerPort |       | string | |
| ingress.subdomain |       | string | |
| requests.cpu |       | string | |
| requests.memory |       | string | |
| limits.cpu |       | string | |
| limits.memory |       | string | |
| vars |       | map | |
| prometheusScraping |       | bool | |
| alwaysPullImages |       | bool | |




!!! note ""

    If you would like to change or add new deployment options, check out the [Master Template](|setup|master-template) guide.