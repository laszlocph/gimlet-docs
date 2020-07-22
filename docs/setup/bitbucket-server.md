# On-premise installation with Bitbucket Server

On this page you will learn how to install the self hosted version of Gimlet.io.

You will use a self hosted Bitbucket Server as source code management system.

## Configure Bitbucket Server

Bitbucket configuration is a long and manual process, please follow the script bellow and make note of your settings. You will use them in configuring Gimlet.

### Create a personal access token

Create a personal access token capable of reading and writing all repositories.

Gimlet will use the token to

- read the `.gimlet.yaml` from each repository
- register webhooks on repositories
- write the gitops repository

We recommend creating a machine account for this purpose.

Navigate to the *Personal access tokens* page in the account settings, and click the *Create a token* button.

![Personal access tokens](./bb-tokens.png)

Set read access for projects and admin access for repositories as shown bellow:

![Personal access tokens](./bb-perms.png)

### Create a key pair

Create a key pair on your server. Gimlet uses the key pair when authorizes Bitbucket users with OAuth.

Generate the private key:

```bash
$ openssl genrsa -out bitbucket.pem 1024
Generating RSA private key, 1024 bit long modulus
....................................++++++
..........++++++
e is 65537 (0x10001)
```


Generate a public key:

```bash
$ openssl rsa \
  -in bitbucket.pem \
  -pubout >> bitbucket.pub
```

### Create an Application Link

Create a Bitbucket Application Link. 

Gimlet uses the link when authorizes Bitbucket users with OAuth.

Navigate the administration panel and click the *Application Links*. Enter your future Gimlet server URL and click *Create new link*.


![Application links](./bb-links.png)

!!! note ""

    *No response was received from the URL you entered - it may not be valid. Please fix the URL below, if needed, and click Continue.*
    
    Since Gimlet is not installed yet, you may see a warning from Bitbucket, click *Continue*


Set the following fields:

|Field | Value|
|-----|--------|
|Application Name| `Gimlet`      |
|Application type  | Generic Application     |
|Service Provider Name  | `Gimlet`     |
|Consumer key  | `OauthKey`     |
|Shared secret  | set a random string, you can generate one with `openssl rand -hex 16`     |
| Request Token URL | your future Gimlet server URL     |
|Access token URL |  your future Gimlet server URL    |
|Authorize URL |  your future Gimlet server URL    |

![Application link](./bb-applink.png)


Once you created the application, it needs to be edited so that you can configure the *Incoming Authentication*.

Please fill out the form using the values specified below and save your changes.

|Field | Value|
|-----|--------|
|Consumer Key  | `OauthKey`     |
|Consumer Name  | `Gimlet`     |
|Public Key  | Paste the contents of the previously generated `bitbucket.pub`     |

## Install Gimlet

You configure Gimlet using environment variables.

### Configuration reference

| Variable | Description | Example |
|-----|--------|-----|
|`HOST`  | Gimlet needs to know its own address | `http://gimlet.mycompany.com` |
|`WEBHOOK_SECRET`  | The secret Gimlet uses to verify incoming webhooks      | Generate one with `openssl rand -hex 16` |
| `BITBUCKET_SERVER_ADDRESS`  | The address Gimlet can access your Bitbucket Server     | `http://bitbucket.mycompany.com` |
| `BITBUCKET_SERVER_USER`  | The personal access token user Gimlet uses to check out repositories, create webhooks, write the gitops repo     | `gimlet` |
| `BITBUCKET_SERVER_TOKEN`  | The personal access token Gimlet uses to check out repositories, create webhooks, write the gitops repo     | `xxxx` |
| `BITBUCKET_SERVER_CONSUMER_KEY`  | To use in the Bitbucket OAuth flow     | `OauthKey` |
| `BITBUCKET_SERVER_CONSUMER_SECRET`  | To use in the Bitbucket OAuth flow     | `yyy` |
| `BITBUCKET_SERVER_PRIVATE_KEY`  | To use in the Bitbucket OAuth flow     | `yyy` |
| `BITBUCKET_SERVER_CONSUMER_SECRET`  | To use in the Bitbucket OAuth flow     | `-----BEGIN RSA PRIVATE KEY----- ...` |
| `BITBUCKET_SERVER_SKIP_VERIFY`  | If your Bitubcket server doesn't use SSL     | `true` |

### Prepare the image pull credentials

With your license comes your registry secret credentials that you can use to pull the Gimlet server image

```bash
export NAMESPACE=xxx

kubectl create secret docker-registry registry.gimlet.io \
    --docker-server=registry.gimlet.io \
    --docker-username=xxx \
    --docker-password=xxx \
    --docker-email=xxx \
    -n $NAMESPACE
```

### Prepare the Bitbucket credentials

Use the values you prepared on the Bitbucket Server UI:

```bash
kubectl create secret generic gimlet-bitbucket \
    --from-literal=BITBUCKET_SERVER_USER=xxx \
    --from-literal=BITBUCKET_SERVER_TOKEN=xxx \
    --from-literal=BITBUCKET_SERVER_CONSUMER_SECRET=xxx \
    -n $NAMESPACE
```

Set the previously generated private key:

```bash
kubectl create secret generic gimlet-bitbucket-privatekey \
    --from-file=bitbucket.pem=/path/to/bitbucket.pem \
    -n $NAMESPACE
```

### Helm chart

Add Gimlet's Helm chart repo:

```bash
helm repo add gimlet-io https://gimlet-io.github.io/helm-chart/
```

Configure Gimlet:

```bash
cat << EOF > values.yaml

config:
  host: https://gimlet.mycompany.com
  bitbucketServer:
    serverAddress: myaddress

ingress:
  annotations:
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
    kubernetes.io/ingress.class: "traefik"
    ingress.kubernetes.io/ssl-redirect: "true"
  hosts:
    - host: gimlet.mycompany.com
      paths:
        - ""
  tls:
    - secretName: tls-gimlet-mycompany-com
      hosts:
        - gimlet.mycompany.com

storageClass: local-path

EOF
```

Finally, install Gimlet:

```bash
helm install gimlet gimlet-io/gimlet -f values.yaml
```

Check the Gimlet server container for errors and logs.

Adjust server parameters with:

```bash
helm upgrade gimlet gimlet-io/gimlet -f values.yaml
```
