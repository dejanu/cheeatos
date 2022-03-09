## Cheatsheet collection

* [Home](#)
* [Ansible](ansible.md)
* [Git](git.md)
* [GCP](index.md)
* [Docker](docker.md)
* [Azure](azure.md)
* <ins>[Terraform](terraform.md)</ins>
* [Helm](helm.md)


### Key features

*  tool for building, changing, and versioning infrastructure safel, that can manage both low-level components such as compute instances, storage, and networking, and high-level components such as DNS entries and SaaS features.

```bash
terraform init # initialize a working directory containing the Terraform configuration files.

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
