## Cheatsheet collection

* [Home](index.md)
* [Ansible](ansible.md)
* [Git](git.md)
* [GCP](index.md)
* [Docker](docker.md)
* [Azure](azure.md)
* <ins>[Terraform](terraform.md)</ins>
* [Helm](helm.md)
* [ElasticSearch](elastic.md)
* [Kubernetes](k8s.md)
* [Istio](istio.md)
* [OIDC](openID.md)
* [PostgreSQL](postgres.md)
* [GitHub Copilot](copilot.md)

---

### Key features

* Tool for building/changing/versioning infrastructure, can manage both low-level components such as compute instances, storage, and networking, and high-level components such as DNS entries and SaaS features.

* Uses a language called HCL - [HashiCorp Configuration Language](https://github.com/hashicorp/hcl).

* Terraform uses **plugins** called **providers** to interface with the resources in the cloud provider.


### Terraform state

* By default is stored in `terraform.tfstate` file or it can be stored remotely e.g. Azure VM


```json
# main.tf file
# store state remotely in Azure VM
terraform {
    backend "azurerm" {
        resource_group_name = "RG_NAME"
        storage_account_name = "STORAGE_ACCOUNT_NAME"
        container_name = "tfstate"
        key = "terraform.tfstate"
    }

    required_version = ">= 0.13"
    required_providers {
    azurerm = {
      source = "hashicorp/azurerm"
      version = "2.49.0"
    }
  }
}
```

* Terraform **backend** used to define the state data files: 
  * By default, Terraform uses a backend called `local`, which stores state as a local file on disk
  * You can configure the backend by adding the `backend` block to your configuration, e.g remote backend:
  
```json
  terraform {
  backend "remote" {
    organization = "example_corp"

    workspaces {
      name = "my-app-prod"
    }
  }
```

* Terraform **state** represents the current state of your infrastructure, while the Terraform **backend** determines where and how this state is stored and accessed.

* Terraform flow:

```bash
terraform init # initialize a working directory containing the Terraform configuration files, it  will create .terraform.lock.hcl to record the provider selections it made above

# validate if the configuration is correct
terraform validate

terraform plan # present you the difference between reality at a given time and config you intend to apply
terraform plan -out maint.tfplan # creates an execution plan to a specified output file

terraform apply # apply the changes required to reach the desired state of the configuration.
terraform apply main.tfplan # apply the plan from file e.g. maint.tfplan

# proposed destroy changes without executing them
terraform plan -destroy -out main.destroy.tfplan
terraform apply main.destroy.tfplan
```

* The `init` command must be called:

  * On any new environment that configures a backend
  * On any change of the backend configuration (including the type of backend)
  * On removing backend configuration completely

### Configuration files

1) Provider block to configure the named provider (multiple providers can exists):

**maint.tf**
```json
# provider block

terraform {
  required_providers {
    google = {
      source = "hashicorp/google"
    }
  }
}


provider "google" {
  version = "3.5.0"

  project = "<PROJECT_ID>"
  region  = "us-central1"
  zone    = "us-central1-c"
}

#compute instance resource

resource "google_compute_instance" "vm_instance" {
  name         = "terraform-instance"
  machine_type = "f1-micro"

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-9"
    }
  }

  network_interface {
    network = google_compute_network.vpc_network.name
    access_config {
    }
  }
}
```


### Action

- If you have a resourge e.g. ResourceGroup create manually in Azure_portal in order to use it you must import it:

```bash
# imported the resource group which was created manually
terraform import azurerm_resource_group.<RG_NAME_LOWERCASE> /subscriptions/<SUBS_ID>/resourceGroups/<RG_NAME>
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