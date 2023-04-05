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
* [OIDC](openID.md)
* [Workshop](workshop.md)

---

<em>"The package manager for k8s, provide users with a better way to manage all the Kubernetes YAML files for a k8s project"</em>

* managing multiple k8s objects it's hard (versioning yaml 😔) - helm allows yml templating
* helm repos: https://artifacthub.io/ and https://bitnami.com/ (VMWare)
* [Kustomize](https://kustomize.io/) (patch k8s obj) `kubectl kustomize .` vs HELM (package manager)
* HELM2 (using **Tiller** pod inside k8s) vs HELM3 which communicates directly with K8s API server (ditch **Tiller** approach)

```bash
## REPOSITORY = chart repository is a location where packaged charts can be stored and share
## CHART = bundle/collection of one or more Kubernetes manifests, a chart is a Helm pacakge.
## RELEASE = instance of a chart running in a k8s cluster.

# add chart repo <REPO_NAME> has been added to your repositories
# downloads an index file that lists all the available charts in the repository
helm repo add [REPO_NAME] [URL]
helm repo add bitnami https://charts.bitnami.com/bitnami

# Make sure we get the latest list of charts
helm repo update 

# list repos
helm repo list
# list charts
helm list -A

# add/list/update repositories: 
helm repo add [REPO_NAME] [URL]
helm repo list
# Make sure we get the latest list of charts
helm repo update
helm search repo <CHART> --versions

# Retrieve a package from a package repository, and download it locally to a tgz file
# Download a chart from a repository and (optionally) unpack it in local directory
helm pull [chart URL | repo/chartname]

# nice flow add repo -> fetch chart to inspect
helm repo add cetic https://cetic.github.io/helm-charts
helm fetch cetic/pgadmin

# install chart very usefull --dry-run
helm install [NAME] [CHART] [flags]

# pick the latest chart version
helm upgrade -i <release_name> <REPO_NAME>  /<CHART> --version 2.12

# create chart
helm create <chart name>

# pick the latest chart and update him : Release "<RELEASE_NAME>  " does not exist. Installing it now.
helm upgrade -i <RELEASE_NAME>  <REPO>/<CHART> --version 2.16

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