---
sidebar_position: 1
---

# Getting started

### Pre-requisites

- Helm `v3.8.0`

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

this command will create a config folder in your home directory `$HOME/.borealis` with the following files:

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

this command will also download