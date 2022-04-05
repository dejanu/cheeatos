## Cheatsheet collection

* [Home](index.md)
* [Ansible](ansible.md)
* [Git](git.md)
* [GCP](gcp.md)
* [Docker](docker.md)
* [Azure](azure.md)
* [Terraform](terraform.md)
* <ins>[Helm](helm.md)<ins>
* [ElasticSearch](elastic.md)
* [Kubernetes](k8s.md)
* [Istio](istio.md)

<em>"The package manager for k8s, provide users with a better way to manage all the Kubernetes YAML files for a k8s project"</em>

```bash
## REPOSITORY = chart repository is a location where packaged charts can be stored and share
## CHART = bundle with one or more Kubernetes manifests

# add chart repo <REPO_NAME> has been added to your repositories
helm repo add <REPO_NAME>  https://artifactory/<REPO_NAME>  --username USER --password PASSWORD

# pick the latest chart version
helm upgrade -i <release_name> <REPO_NAME>  /<CHART> --version 2.12

# add/list/update repositories:
helm repo add [NAME] [URL]
helm repo list
helm repo update
helm search repo <CHART> --versions

# create chart
helm create <chart name>

# pick the latest chart and update him : Release "<RELEASE_NAME>  " does not exist. Installing it now.
helm upgrade -i <RELEASE_NAME>  <REPO>/<CHART> --version 2.16

# add/install chart repo:
helm repo add bitnami https://charts.bitnami.com/bitnami

# Make sure we get the latest list of charts
helm repo update 

# Once repo is installed, list the charts:
helm search repo bitnami

# Install chart:
helm install bitnami/mysql --generate-name
helm install <realease_name> bitnami/rabbitmq
```

* When installing Helm, make sure you're installing version 3. Version 2 still works, but it needs a server-side component called **Tiller**, which ties your helm installation to a single cluster. Helm 3 removed this need with the addition of several CRDs, but it's not supported in all Kubernetes versions.

* Helm chart **repositories** [Artifact Hub](https://artifacthub.io/packages/search?kind=0):

* RabbitMQ chart:

```bash

# add repo
helm repo add bitnami https://charts.bitnami.com/bitnami

# install the chart, with given deployment name or auto generate deployment name
helm install <realease_name> bitnami/rabbitmq
helm install bitnami/rabbitmq --generate-name

# create k8s namespace
kubectl create namespace rabbit
helm install <release_name> bitnami/rabbitmq --namespace rabbit

# list installed charts
helm list

# delete/uninstall chart
helm delete <release_name>

# get objects from namespace
kubectl get deployments,pods,services --namespace rabbit
```
---

```bash
                    ___ _____
                   /\ (_)    \
                  /  \      (_,
                 _)  _\   _    \
                /   (_)\_( )____\
                \_     /    _  _/
                  ) /\/  _ (o)(
                  \ \_) (o)   /
                   \/________/         @dejanualex
```