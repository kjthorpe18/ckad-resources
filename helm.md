# Helm
Helm is a package manager for Kubernetes that simplifies the process of deploying and managing applications on a Kubernetes cluster.

The Helm documentation at https://helm.sh/docs/ is allowed during the CKAD exam.

## Helm Concepts
```
Templates:      Resource definition files with variables
values.yaml:    A file containing variables for configuring resources in a release
Chart:          Templates + values.yaml file
Release:        An instance of a chart running in a K8s cluster (like the image-container relationship)
```

## Installing Helm
https://helm.sh/docs/intro/install/

## Helm commands
```sh
helm create <name>              # Create a basic chart

helm show values <repo/app>     # Show a chart's values

# releasename is what you are naming this release
# chart can be a path to a local chart, or `repo/chart`
helm install <releasename> <chart>
--set <key=value>               # Override a value in the chart
--set-string <key=value>        # Override a value in the chart as a string
--values <values.yaml>          # specify values
-f <values.yaml>                # specify a values file

# Ex. $ helm install --set replicaCount=4 myredis ./redis

helm upgrade <releasename> <repo/chart> # Upgrade a release's version

helm rollback <releasename> [options]

helm uninstall <releasename> [options]

helm pull --untar <chart>       # download but don't install a chart

helm list                       # List installed releases

helm repo add <repo> <url>      # Add a repository
helm repo list                  # list current repositories

helm search hub <chart>         # Search releases in the Artifact Hub
helm search repo <chart>        # Search releases in all local repositories
```