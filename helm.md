## Cheatsheet collection

* [Home](#)
* [Ansible](ansible.md)
* [Git](git.md)
* [GCP](index.md)
* [Docker](docker.md)
* [Azure](azure.md)
* [Terraform](terraform.md)
* <ins>[Helm](helm.md)<ins>

<em>"The package manager for k8s Provide users with a better way to manage all the Kubernetes YAML files for a k8s project"</em>

```bash

## CHART = bundle with one or more Kubernetes manifests.

# create chart
helm create <chart name>

# add chart repo <REPO_NAME> has been added to your repositories
helm repo add <REPO_NAME>  https://artifactory/<REPO_NAME>  --username USER --password PASSWORD

# pick the latest chart version
helm upgrade -i <release_name> <REPO_NAME>  /<CHART> --version 2.12

# search in repo
helm repo list
helm repo update
helm search repo <CHART> --versions

 # pick the latest chart and update him : Release "<RELEASE_NAME>  " does not exist. Installing it now.
helm upgrade -i <RELEASE_NAME>  <REPO>/<CHART> --version 2.16
```

* When installing Helm, make sure you're installing version 3. Version 2 still works, but it needs a server-side component called **Tiller**, which ties your helm installation to a single cluster. Helm 3 removed this need with the addition of several CRDs, but it's not supported in all Kubernetes versions.

* Helm chart **repositories** [Artifact Hub](https://artifacthub.io/packages/search?kind=0)
```bash
# add/install chart repo:
helm repo add bitnami https://charts.bitnami.com/bitnami

# Make sure we get the latest list of charts
helm repo update 

# Once repo is installed, list the charts:
helm search repo bitnami

# Install chart:
helm install bitnami/mysql --generate-name
```

