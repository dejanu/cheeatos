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
* [PostgreSQL](postgres.md)
* [GitHub Copilot](copilot.md)

---

<em>"The package manager for k8s, provide users with a better way to manage all the Kubernetes YAML files for a k8s project"</em>

* Managing multiple k8s objects it's hard (versioning yaml 😔) - helm allows yml templating

* Helm chart contents:

![char content](https://github.com/dejanu/cheetcity/blob/gh-pages/src/helm_chart_structure.PNG?raw=true)

* Helm repos: [Artifact Hub](https://artifacthub.io/packages/search?kind=0) and https://bitnami.com/ (VMWare)
* [Kustomize](https://kustomize.io/) (patch k8s obj) `kubectl kustomize .` vs HELM (package manager)
* HELM2 (using **Tiller** pod inside k8s) vs HELM3 which communicates directly with K8s API server (ditch **Tiller** approach)

```bash
## REPOSITORY = HTTP server where packaged charts can be stored and shared
## CHART = bundle/collection of one or more Kubernetes manifests, a chart is a Helm pacakge
## RELEASE = instance of a chart running in a k8s cluster

# add chart repo <REPO_NAME> has been added to your repositories
# downloads an index file that lists all the available charts in the repository
helm repo add [REPO_NAME] [URL]
helm repo add bitnami https://charts.bitnami.com/bitnami

# add/list/update repositories (make sure we get the latest list of charts)
helm repo add [REPO_NAME] [URL]
helm repo list
helm repo update
helm search repo <REPO>/<CHART> --versions

# Retrieve a package from a package repository, and download it locally to a tgz file
# Download a chart from a repository and (optionally) unpack it in local directory
helm pull [chart URL | repo/chartname]

# nice flow add repo -> fetch chart to inspect
helm repo add cetic https://cetic.github.io/helm-charts
# download and extract the chart
helm fetch cetic/pgadmin --untar

# list  all of the releases in all namespaces
helm list -A

# impersonate user cluster-admin
helm list -A --kube-as-user cluster-admin

# re-read chart configuration information
helm status <release_name>

# inspect chart info
helm show all <CHART>

# inspect chart values
helm show values <CHART>

# fetch metadata for a current release
helm get metadata RELEASE_NAME [flags]

# override chart configuration values
helm install --set replicaCount=2  <release_name> <CHART>
helm install -f values.yaml <release_name> <CHART>

# install chart very usefull --dry-run
helm install [NAME] [CHART] [flags]

# vanilla install chart
helm install bitnami/rabbitmq --generate-name
helm install <release_name> bitnami/rabbitmq

# install and create namespace
helm install kube-green kube-green/kube-green --namespace kube-green --create-namespace

# install latest chart and updates it: Release "<RELEASE_NAME> " does not exist. Installing it now.
# upgrade a release
helm upgrade -i <RELEASE_NAME>  <REPO>/<CHART> --version 2.16

# create chart
helm create <chart name>
```

* When installing Helm, make sure you're installing version 3. Version 2 still works, but it needs a server-side component called **Tiller**, which ties your helm installation to a single cluster. Helm 3 removed this need with the addition of several CRDs, but it's not supported in all Kubernetes versions.


* RabbitMQ flow:

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
helm repo list
helm search repo bitnami
helm fetch bitnami/rabbitmq --untar

# create k8s namespace
kubectl create ns rabbit

# helm install <release_name> bitnami/rabbitmq
helm install testrabbit ./rabbitmq/ --namespace rabbit

# auto generate name 
helm install bitnami/rabbitmq --generate-name --namespace rabbit

# list charts and view charts history
helm list

# delete/uninstall chart
helm delete <release_name>

# get helmrealease for a namespace
kubectl -n <namespace> get hr

# get helm values for a chart
helm -n <namespace> get values <chart_name>
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