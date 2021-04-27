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
## Storage

- Storage bucket: basic containers that hold your data
**Create storage bucket**
```bash
gsutil mb -p <PROJECT_ID> gs://<BUCKET_NAME>
```
**Copy from/to Bucket**

```bash
gsutil cp -r gs://<BUCKET_NAME> .
gsutil cp <FILE> gs://<BUCKET_NAME>
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

**create VM with startup script**
```bash
gcloud compute instances create www1 \
  --image-family debian-9 \
  --image-project debian-cloud \
  --zone us-central1-a \
  --tags network-lb-tag \
  --metadata startup-script="#! /bin/bash
    sudo apt-get update
    sudo apt-get install apache2 -y
    sudo service apache2 restart
    echo '<!doctype html><html><body><h1>www1</h1></body></html>' | tee /var/www/html/index.html"
```

**ssh into VM**
```bash
gcloud compute ssh <VM_NAME> --zone <ZONE>
```
