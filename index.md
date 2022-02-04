## Cheatsheet collection

* [Home](#)
* [Ansible](ansible.md)
* [Git](git.md)
* <ins>[GCP](index.md)<ins>
* [Docker](docker.md)
* [Azure](azure.md)
* [Terraform](terraform.md)

### GCP

* [Download SDK](https://cloud.google.com/sdk/docs/install#linux)



## gcloud SDK
```python
# Cloud Shell is a VM with gcloud sdk
```
```bash
# install gcloud sdk
sudo apt-get install google-cloud-sdk

gcloud compute project-info describe --project $(gcloud config get-value core/project)
# list components e.g.: gsutil, kubectl
gcloud components list

# grant/revoke authorization to Cloud SDK
gcloud auth login
gcloud auth revoke
```

***

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

Region vs Zone: 
![alt text](https://github.com/dejanu/cheetcity/blob/gh-pages/src/zone_region.png?raw=true)


```bash
gcloud config get-value compute/region
gcloud config get-value compute/zone

gcloud config set compute/region us-east1
gcloud config set compute/zone us-east1-d
```
***

**Enable APIs**
```bash
# enable texttospeech API
gcloud services enable texttospeech.googleapis.com
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
gsutil -m cp -r gs://spls/gsp233/* .
```

***

## Cloud Source Repos

**Create repo**
```bash
# create source repo <REPO_NAME> @ https://source.developers.google.com/p/$PROJECT/r/s<REPO_NAME>
gcloud source repos create <REPO_NAME>

# initialize a repo
git init
git remote add origin [your-repository]
git pull [your-repository]
```

**Clone repo**
```bash
# clone repo
gcloud source repos clone <REPO_NAME> --project=<PROJECT_ID>
```

***

## Compute Engine (IaaS)


**list all VM instances in a project**
```bash
#by default instances from all zones are listed
gcloud compute instances list
```
**create VM**
```bash
# create compute instances Linux (Debian,Ubuntu, Suse, Red Hat, CoreOS) and Windows Server, on Google infrastructure
gcloud compute instances create <VN_NAME> --machine-type <MACHINE_TYPE> --zone <ZONE>

gcloud compute instances create gcelab2 --machine-type n1-standard-2 --zone us-central1-c

# create VM with tags
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

**Create Virtual Private Cloud (VPC) network and subnetworks**

```bash
# Create VPC griffin-dev-vpc with 2 subnets griffin-dev-wp and griffin-dev-mgmt
gcloud compute networks create griffin-dev-vpc --subnet-mode=custom
gcloud compute networks subnets create griffin-dev-wp --network=griffin-dev-vpc --range=192.168.16.0/20
gcloud compute networks subnets create griffin-dev-mgmt --network=griffin-dev-vpc --range=192.168.32.0/20
```

**Create a bastion host (jump server) and configure NIC to subnet**
```bash
gcloud compute instances create bastion-vm --machine-type=n1-standard-1 --subnet=griffin-dev-mgmt
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

# get namespace
kubectl config view | grep namespace
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



export SERVICE_PORT=$(kubectl get --namespace <NAMESPACE> -o jsonpath="{.spec.ports[0].port}" services <SERVICE>)
export SERVICE_IP=$(kubectl get svc --namespace <NAMESPACE> <SERVICE> -o jsonpath='{.status.loadBalancer.ingress[0].ip}')

# get nodes
kubectl get nodes -o wide

# check pod running images
kubectl get pods -o=jsonpath='{range .items[*]}{"\n"}{.metadata.name}{":\t"}{range .spec.containers[*]}{.image}{", "}{end}{end}' |sort
```
***

## App Engine (PaaS)

```python
# App Engine standard environment is based on container instances (preconfigured with one of several available runtimes Java 7/8, Python 2.7, Go and PHP)) running on Google's infrastructure.
# App Engine is a serverless compute platform that is fully managed to scale up and down as workloads demand
```
**Deploy golang app to AppEngine**

* [Golang helloword app](https://github.com/GoogleCloudPlatform/golang-samples/tree/master/appengine/go11x/helloworld)

```bash
# install component
gcloud components install app-engine-go

# deploy  app's code and configuration to the App Engine server
gcloud app deploy

#  app.yml to define the runtime: go115 
gcloud app deploy app.yaml --project $PROJECT_ID -q

# open the current app in a web browser
gcloud app browse
```
```yaml
# app.yaml content
runtime: go113

handlers:
- url: /.*
  secure: always
  script: auto

env_variables:
  PORT: "8080"
```
***

## Cloud Functions (FaaS)

```bash
# event-driven serverless compute platform
```
**index.js as cloud function**
```js
/**
* Background Cloud Function to be triggered by Pub/Sub.
* This function is exported by index.js, and executed when
* the trigger topic receives a message.
*
* @param {object} data The event payload.
* @param {object} context The event metadata.
*/
exports.helloWorld = (data, context) => {
const pubSubMessage = data;
const name = pubSubMessage.data
    ? Buffer.from(pubSubMessage.data, 'base64').toString() : "Hello World";

console.log(`My Cloud Function: ${name}`);
};
```

**Create storage bucket and deploy function to bucket**
```bash
gsutil mb -p <PROJECT_ID> gs://<BUCKET_NAME>


gcloud functions deploy helloWorld \
  --stage-bucket <BUCKET_NAME> \
  --trigger-topic hello_world \
  --runtime nodejs8

#verify function status
gcloud functions describe helloWorld

# test function/ check logs
DATA=$(printf 'Hello World!'|base64) && gcloud functions call helloWorld --data '{"data":"'$DATA'"}'
gcloud functions logs read helloWorld
```
