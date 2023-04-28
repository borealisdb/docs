---
sidebar_position: 1
---

# Getting started

### Pre-requisites

- Helm `v3.8.0`
- A kubernetes cluster with an ingress (Ex: nginx-ingress) deployed and TLS, metrics-server is optional but recommended
- An IDP provider with SSO OIDC (Ex: Auth0)

## Borealis CLI

To use borealisdb you don't need the CLI, but it will help to get you up and running quicker.

### Download and install

You can find all the cli binaries here: https://github.com/borealisdb/cli/releases

```shell
wget -qO- https://github.com/borealisdb/cli/releases/download/v0.0.7/borealisdb_0.0.7_linux_x86_64.tar.gz | sudo tar -xvz -C /usr/local/bin/ borealisdb
```

Check that the cli has been installed correctly

```shell
$ borealisdb
Borealis CLI: v0.0.7
```

### Initialize the cli

```shell
borealisdb init --host <https://yourhost>
```

this command will download the latest helm chart needed for the deployment, you can find the charts
here: https://borealisdb.github.io/charts/,
and create a config folder in your home directory `$HOME/.borealis` with the following files:

- `config.ini`
- `credentials.ini`

for now there is no need to modify them

## Create a new project

When you create a project the cli will scaffold the following directory structure:

```shell
borealis/
├── development/
│   ├── your-cluster-name/
│   │   ├── accounts.yaml
│   │   ├── cluster.yaml
│   │   └── secrets.yaml
│   └── infrastructures
│       ├── secrets.yaml
│       └── values.yaml
└── production/
    ├── your-cluster-name/
    │   ├── accounts.yaml
    │   ├── cluster.yaml
    │   └── secrets.yaml
    └── infrastructures/
        ├── secrets.yaml
        └── values.yaml
```

Create the project:

```shell
borealisdb create --cluster-name <your cluster> --host <https://yourhost>
```

### Configure your new project

- You need to fill your infrastructures secrets located in `borealis/<environment>/infrastructures`:

    ```yaml
    apiVersion: v1
    kind: Secret
    metadata:
      name: borealis-secrets
    type: Opaque
    data:
      oidcProviderID: "" # All your IDP SSO configuration
      oidcProviderName: ""
      oidcProviderClientID: ""
      oidcProviderClientSecret: ""
      oidcProviderClientIssuerUrl: ""
      clientSecret: "" # This is needed for the identity broker, you can auto-generate one yourself
      clusterTokenSecret: "" # This the secret needed to encrypt cluster tokens, you can as well generate one yourself
    ```

:::note

Borealisdb uses native Kubernetes secrets which are not securely encrypted.
We suggest you use something like [External secrets operator](https://external-secrets.io/v0.8.1/), or others.
If you don't want to use any third party solution you should configure a strict RBAC rules for this resource.

:::

- For your IDP you need to whitelist the following callback: `https://<yourhost>/borealis/identity/callback`

- Add your email to `accounts.yaml` in `borealis/<environment>/<your cluster name>`; it should be the email belonging to your IDP
  ```yaml
    apiVersion: borealisdb.io/v1
    kind: BorealisClusterAccount
    metadata:
      name: <your-cluster-name>-accounts
      labels:
        clusterName: <your-cluster-name>
    spec:
    accounts:
      - email: <your-email@company.com>
        role: admin
  ```
  `BorealisClusterAccount` is a CRD that helps you manage permissions and role associations in the cluster, adding your email to the admin
  role will grant you access to the cluster

- In the `values.yaml`, change your storage class name if you are not using the `standard` one

    ```yaml
    storage:
      storageClassName: gp2 # Check the name from your provider
    ```
  The full configuration `values.yaml` can be found [here](https://github.com/borealisdb/charts/blob/master/borealis/values.yaml)

If you are just testing this is all you need to configure

## Deploy the infrastructures

```shell
borealisdb deploy --environment <env>
```
This command will deploy the borealis infrastructures and the borealis operator, if enabled (true by default) the default backup system.

## Login with the CLI

```shell
borealisdb login
```

This will get a token in which will give you access to the borealisdb API and cluster

## Deploy the cluster

Now you are ready to deploy the PostgreSQL cluster:

```shell
borealisdb cluster deploy --cluster-name <yourcluster> --environment <env>
```

## Connect to the cluster

```shell
borealisdb cluster connect --cluster-name <yourcluster> --userrole admin
```

:::note

In order to run this command successfully you need to first be login with the cli: `borealisdb login`

:::

This command will do a lot for you and you can do the same without the cli:

- Download the cluster certificates: 
  ```shell
  kubectl get secrets <yourcluster>-tls -o 'jsonpath={.data.root\.crt}' | base64 -d > root.crt
  ```
- Get a cluster token
  - With the cli: `borealisdb cluster token --cluster-name <yourcluster>`
  - Via curl:
    ```shell
    curl -X POST -H 'Authorization: Bearer ...'  https://<yourhost>/borealis/clusters/oauth2/token
    ```
- Port forward to the master (or replica) service:
  ```shell
  kubectl port-forward <endpoint> 5432:5432 
  ```
- Login with psql:
  ```shell
  psql "sslmode=verify-ca sslrootcert=$HOME/.borealis/root.crt host=localhost user=admin port=5432 dbname=postgres"
  ```
  


# Explore your new cluster

You can now login into the Borealis console, navigate to `https://<yourhost>/borealis/monitoring`, it will redirect you to your IDP login
page if you are not logged in yet.
You should see your cluster appearing in the cluster list, click on it to explore the metrics and logs.
Enjoy!

