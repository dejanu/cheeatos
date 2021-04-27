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


**Create storage bucket**
```bash
# storage bucket: basic containers that hold your data
gsutil mb -p <PROJECT_ID> gs://<BUCKET_NAME>
```

**Copy from/to Bucket**
```bash
gsutil cp -r gs://<BUCKET_NAME> .
gsutil cp <FILE> gs://<BUCKET_NAME>
```

***

## Compute Engine (Iaas)

**create VM**
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

***

## GKE (K8s Engine)

**Create k8s cluster**
```bash
## K8s environment consists of multiple machines (Compute Engine instances) grouped to form a container cluster
gcloud container clusters create [CLUSTER-NAME]

# example of cluster with 2 worker nodes
gcloud container clusters create hello-world \
                --num-nodes 2 \
                --machine-type n1-standard-1 \
                --zone us-central1-a
```

**List k8s clusters**
```bash
gcloud container clusters list
```
**Fetch cluster auth data for kubeconfig**
```bash
gcloud container clusters get-credentials [CLUSTER-NAME]
```

**Create deployment object IMPERATIVE for deploying stateless applications**
```bash
kubectl create deployment <DEPLOYMENT_NAME> --image=gcr.io/<PROJECT_ID</<IMAGE_NAME:TAG>
kubectl create deployment hello-server --image=gcr.io/google-samples/hello-app:1.0
```

**Create service**
```bash
# provide public IP and in order to access nodes from external 
kubectl expose deployment hello-node --type="LoadBalancer" --port=8080

```

**Create deployment and service DECLARATIVE**
```bash
# based on 
kubectl create -f deployment.yaml
kubectl create -f service.yaml
```

**Create k8s namespace**
```bash
# NAMESPACE = objects which partition a single K8s cluster into multiple virtual clusters
kubectl create ns <NAMESPACE>
```

**Delete deployment**
```bash
gcloud deployment-manager deployments delete <DEPLOYMENT_NAME> --delete-policy=DELETE
```

**Restart deployment**
```bash
kubectl rollout restart deployment/<DEPLOYMENT_NAME>

# interactive rollout
kubectl edit deployments/<DEPLOYMENT_NAME>
```

**Scale deployment (replicasets)**
```
# from 1 to 3
kubectl scale --current-replicas=1 --replicas=3 deployment/<DEPLOYMENT_NAME>
```
**Get replicasets/nodes/pods/services/deployments**
```bash
kubectl cluster-info
kubectl get pods
kubectl get svc
kubectl get replicasets
kubectl get rs
kubectl get deployments

# check pod running images
kubectl get pods -o=jsonpath='{range .items[*]}{"\n"}{.metadata.name}{":\t"}{range .spec.containers[*]}{.image}{", "}{end}{end}' |sort
```

***
