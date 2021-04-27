## Cheatsheet collection



### GCP

**List/Set gcp project**

```bash
gcloud projects create <PROJECT_ID> --organization=<ORGANIZATION_ID>
gcloud config list project
gcloud config set project <my-project>
```
**Get project ID**
```bash
# export project id as environment variable
export PROJECT_ID=$(gcloud config get-value core/project)
export PROJECT_ID=$(gcloud info --format='value(config.project)')
export PROOJECT_ID=$(gcloud config list --format 'value(core.project)')
```

**Get/Set compute zone/region**
```bash
gcloud config get-value compute/region
gcloud config get-value compute/zone

gcloud config set compute/zone us-east1-d
gcloud config set compute/region us-east1
```

***


## Compute Engine (Iaas)

**create VM's**
```bash
# create compute instances Linux (Debian,Ubuntu, Suse, Red Hat, CoreOS) and Windows Server, on Google infrastructure
gcloud compute instances create <VN_NAME> --machine-type <MACHINE_TYPE> --zone <ZONE>

gcloud compute instances create gcelab2 --machine-type n1-standard-2 --zone us-central1-c
gcloud compute instances create lamp-1-vm --machine-type n1-standard-2 --zone=us-central1-a --tags="Allow HTTP traffic"
```
