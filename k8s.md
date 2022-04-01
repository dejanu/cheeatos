* [Home](#)
* [Ansible](ansible.md)
* [Git](git.md)
* [GCP](index.md)
* [Docker](docker.md)
* [Azure](azure.md)
* [Terraform](terraform.md)
* [Helm](helm.md)
* [ElasticSearch](elastic.md)
* <ins>[Kubernetes](k8s.md)<ins>
* [Istio](istio.md)

### Debugging pods:

```bash

# list events
kubectl describe pod/<pod_name> -n <namespace>
kubectl describe pod <pod_name> -n <namespace>

# spin up new pod with desired image
kubectl run -ti kali --image=kalilinux/kali-rolling

# delete the kali pod
kubectl delete -n default pod kali

# spin up a shell inside the pod
kubectl exec -n <namespace> -it <pod_name> -- bash

# execute cmd:e.g. curl from one pod to another
kubectl exec -t -n <namespace> <pod_name> -- curl -I http://<another_pod_ip>:3030/metrics

# get pod logs from all namespaces
$for n in $(kubectl get ns | awk 'FNR>1 {print $1}');do kubectl get pods -n $n;done
kubectl logs <pod_name> --all-containers -n <namespace_name>

# get logs  from a specific pod
kubectl -n <namespace> logs <pod_name> --tail 200

# check pod running images
kubectl get pods -o=jsonpath='{range .items[*]}{"\n"}{.metadata.name}{":\t"}{range .spec.containers[*]}{.image}{", "}{end}{end}' |sort
```
 ***
### Nodes operations:

```bash
# get nodes labes
kubectl get nodes --show-labels
kubectl get nodes -o wide

# describe node and check conditions field describes the status of all Running nodes: True if pressure
kubectl describe node <insert-node-name-here>

 Conditions:
  Type                 Status  Message
  ----                 ------  -----------------
  NetworkUnavailable   False   Weave pod has set this
  MemoryPressure       False   kubelet has sufficient memory available
  DiskPressure         False   kubelet has no disk pressure
  PIDPressure          False   kubelet has sufficient PID available
  Ready                True    kubelet is posting ready status. AppArmor enabled
  
# kubectl drain to safely evict all of your pods from a node before you perform maintenance on the node (e.g. kernel upgrade, hardware maintenance)
kubectl drain <node>
kubectl drain node_name --ignore-daemonsets

# Cordon the node; this means marking the node itself as unplannable so that new pods are not arranged on the node. 
# Kubectl contains a command named cordon that permits us to create a node unschedulable
kubectl uncordon <node_name>
```
 ***
### kubectl configs:

 ```bash
# autocompletion 
source <(kubectl completion bash)

# Displays the current-context/displays merged kubeconfig settings or a specified kubeconfig file.
kubectl config current-context
kubectl config view

# Sets/Unset an individual value in a kubeconfig file
kubectl config set
kubectl config unset

# Sets a cluster entry in kubeconfig
kubectl config set-cluster

# Sets a context entry in kubeconfig
kubectl config set-context
# Sets a user entry in kubeconfig
kubectl config set-credentials

# Sets the current-context in a kubeconfig file
kubectl config use-context
 ```
 ***
### Get k8s objects:
```bash
# replicasets/nodes/pods/services/deployments/daemonsets/statefulsets/cronjobs
kubectl cluster-info
kubectl get pods
kubectl get svc
kubectl get replicasets
kubectl get rs
kubectl get deployments

# get namespaces
kubectl config view | grep namespace
kubectl get ns

# NAMESPACE = objects which partition a single K8s cluster into multiple virtual clusters
kubectl create ns <namespace>
```
 ***
### Deployment ops:
```bash

# interactive rollout
kubectl edit deployments/<deployment_name>
```
 ***
### Services:

```bash
# get services
kubectl -n <namespace> get svc

# get pod port
kubectl get pod <pod_name> --template='{{(index (index .spec.containers 0).ports 0).containerPort}}{{"\n"}}' -n <namespace>

# tunnels the traffic from a specified port at your local host machine to the specified port on the specified pod
kubectl port-forward pods/<pod_name> <port_no>:<port_no>

```
  ***

### 101 stuff:

**probes**:
- Liveness probe checks the container health and if it fails it restarts the container (kubelet uses liveness probes to know when to restart a container)

- Readiness Probe check if a POD is ready to serve trafic (kubelet uses readiness probes to know when a container is ready to start accepting traffic. A Pod is considered ready when all of its containers are ready)

**requests_and_limits**:

- CPU <k8s_CPU's> and memory <bytes> are each a resource type and form compute resources, which can be requested, allocated, and consumed by containers.
- CPU resource units **limits** and **requests** are measured in CPU unints.
- **requests** is what the container is guaranteed to get.
- **limits** is what the container is allowed to use, and is restricted to go above limits.

**CRD**:

- A RESOURCE is an endpoint in the Kubernetes API that stores a collection of API objects of a certain kind
- CUSTOM RESOURCES are extensions of the Kubernetes API.
- Extend the k8s API with CRD -> custom resource (CRD or Aggregated API)
```bash

# kubectl get CRD
kubectl get crd -n <namespace>
kubect get <customresource_name> -n <namespace>
kubectl describe  <customresource_type> <customresource_name> -n <namespace>

```
