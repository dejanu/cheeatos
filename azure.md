## Cheatsheet collection

* [Home](index.md)
* [Ansible](ansible.md)
* [Git](git.md)
* [GCP](gcp.md)
* [Docker](docker.md)
* <ins>[Azure](azure.md)<ins>
* [Terraform](terraform.md)
* [Helm](helm.md)
* [ElasticSearch](elastic.md)
* [Kubernetes](k8s.md)
* [Istio](istio.md)
* [OIDC](openID.md)

---

* Check Azure status: https://status.azure.com/en-us/status

* Commands:

```bash
# get account details e.g. subscribtion,tenant id 
az account show
az account show -o table

# list subscriptions
az account list --output table

# change subscription
az account set -s NAME_OR_ID
az account set -s <subscription_guid>

# get tenant ID and subscriptions
az account list -o table --all --query "[].{TenantID: tenantId, Subscription: name, Default: isDefault}"

# list resource groups from a subscription based on query
az group list --query "[?location=='westeurope']" -o table
az group list --query "[?name=='RESOURCE_GROUP_NAME']" -o table
az group list -o table

# check resource group components aka resources
az group show --resource-group RESOURCE_GROUP_NAME
az resource list --query "[?resourceGroup=='RESOURCE_GROUP_NAME']"

# list service principals
az ad sp list --query "[].{id:id,name:displayName}" --show-mine

# create service principal app_name in AAD + https://stackoverflow.com/questions/55457349/service-principal-az-cli-login-failing-no-subscriptions-found
export mySubscriptionID="3e053c67-dd5c-4904-8ecb-5af26418d771"
az ad sp create-for-rbac --name <service_principal> --role contributor --scopes /subscriptions/$mySubscriptionID

# login using sp - username for a service principal is its Application is (client) ID
az login --output json --password <service_principal_password> --service-principal --tenant <AAD_tenant> --username <service_principal>

# reset credentials for service principal will output to STDOUT the new credentials
az ad sp credential reset --name <service_principal>

# assign role to SVP
az role assignment create --assignee "object_id" --role "contributor"
  
# check AKS/K8s node pool
az aks show --resource-group RESOURCE_GROUP_NAME --name AKS_CLUSTER_NAME --query agentPoolProfiles

# scale AKS/k8s cluster: set 
az aks scale --resource-group RESOURCE_GROUP_NAME --name AKS_CLUSTER_NAME --node-count 4 --nodepool-name NODEPOOL_NAME

# get k8s cluster credentials .kube/config
az aks get-credentials --resource-group <resourge_group_name> --name <cluster-name>

# get supported k8s version by region
az aks get-versions --location westeurope
```
---

* Azure Management Groups - containers that help you manage access, policy, and compliance for multiple subscriptions
* Azure Subscriptions -  authenticates and authorizes user to use resources, and a subscription is linked to an Azure account, which in turn is an identity in Azure Active Directory (AD). Hence, a subscription is an agreement between an organization and Microsoft to use resources, for which charges are either paid on a per-license basis or a cloud-based, resource-consumption basis.
* Azure Resources Groups - logical collections of virtual machines, storage accounts, virtual networks, web apps, databases, and/or database servers
* Service principal - identity created for use with applications, hosted services, and automated tools to access Azure resources instead of havving apps signs is as a fully privileged user.

* Billing account - Subscription - Resource Group (A billing account can contain multiple subscriptions within it to help isolate and organize how payments are organized. Subscriptions contain multiple resource groups, which are collections of related resources, such as compute, storage, and network resources within the same application)


Azure subscription and resource groups:

![alt text](https://github.com/dejanu/cheetcity/blob/gh-pages/src/azure_hierachy.png?raw=true)

---

* Wrapper script:

```bash
# check if user is logged in azure cli
if [ -z "$(az account show)" ]; then
    echo "Please login to azure cli"
    az login --use-device-code
else
    echo "Logged in to azure cli with the following account:"
    az account show -o table
fi
separator_stuff="\e[1;32m ===============================================================\e[0m\n"
# list available subscriptions
echo -e "$separator_stuff Available subscriptions:"
az account list -o table --all

# select and set subscription using subscription_id variable
echo -e "$separator_stuff Select subscriptionId :"
read subscription_id
az account set --subscription $subscription_id

# list all resource groups in the selected subscription
echo -e "$separator_stuff Available RESOURCE GROUPS in the selected subscription:"
#az group list -o table --query "[?contains(id, '$subscription_id')]"
az group list -o table

# select and set resource group using resource_group variable
echo -e "$separator_stuff Select resource group :"
read resource_group
echo -e "$separator_stuff Available RESOURCES in the $resource_group RESOURCE GROUP:"
az resource list --query "[?resourceGroup=='$resource_group']" -o table
```
---

* [Azure AKS upgrade](https://faun.pub/tale-of-a-kubernetes-upgrade-7a08e5d5528a)

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