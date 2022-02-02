## Cheatsheet collection

* [Home](#)
* [Ansible](ansible.md)
* [Git](git.md)
* [GCP](index.md)
* [Docker](docker.md)
* <ins>[Azure](azure.md)<ins>
* [Terraform](terraform.md)

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

# list resource groups based on query
az group list --query "[?location=='westeurope']" -o table
az group list --query "[?name=='NAME_OF_RESOURCE_GROUP']" -o table

# check resource group components aka resources
az group show --resource-group PHISynergyRG1
az resource list --query "[?resourceGroup=='NAME_OF_RESOURCE_GROUP']"
```
* Azure Management Groups - containers that help you manage access, policy, and compliance for multiple subscriptions
* Azure Subscriptions -  authenticates and authorizes user to use resources, and a subscription is linked to an Azure account, which in turn is an identity in Azure Active Directory (AD). Hence, a subscription is an agreement between an organization and Microsoft to use resources, for which charges are either paid on a per-license basis or a cloud-based, resource-consumption basis.
* Azure Resources Groups - logical collections of virtual machines, storage accounts, virtual networks, web apps, databases, and/or database servers

