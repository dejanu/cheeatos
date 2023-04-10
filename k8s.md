## Cheatsheet collection

* [Home](index.md)
* [Ansible](ansible.md)
* [Git](git.md)
* [GCP](gcp.md)
* [Docker](docker.md)
* [Azure](azure.md)
* [Terraform](terraform.md)
* [Helm](helm.md)
* [ElasticSearch](elastic.md)
* <ins>[Kubernetes](k8s.md)<ins> -> [k8s_objects](k8s_resources.md) 
* [Istio](istio.md)
* [OIDC](openID.md)
* [PostgreSQL](postgres.md)

---

### Debugging pods:

```bash
# flags
# allowed formats are: custom-columns,custom-columns-file,go-template,go-template-file,json,jsonpath,jsonpath-file,name,template,templatefile,wide,yaml
kubectl get pods -A (--all-namespaces )
kubectl get pods --show-labels
kubectl get pods -w (--watch)
kubectl get pods -o json

# list events
kubectl describe pod/<pod_name> -n <namespace>

# running debug pods, svc in the same namespace are resolvable by DNS 
kubectl run -ti kali --image=kalilinux/kali-rolling
kubectl delete -n default pod kali
kubectl run -n <namespace> mybusyboxcurl --image yauritux/busybox-curl -it -- sh

# spin up a shell inside the pod
kubectl exec -n <namespace> -it <pod_name> -- bash

# execute cmd:e.g. curl from one pod to another
kubectl exec -t -n <namespace> <pod_name> -- curl -I http://<another_pod_ip>:3030/metrics

# get pod logs from all namespaces
$for n in $(kubectl get ns | awk 'FNR>1 {print $1}');do kubectl get pods -n $n;done

# logs for for a specific RESOURCE: deployment is specified and that deployment has multiple pods such as a ReplicaSet
# then only one of the pods logs will be returned
kubectl -n <namespace> logs deployment/<deploymet_name> --all-containers=true --since 10m

# get logs (no_lines) from a specific POD
kubectl -n <namespace> logs <pod_name> --tail 200 --timestamps=true

# check pod running images
kubectl get pods -A -o=jsonpath='{range .items[*]}{"\n"}{.metadata.name}{":\t"}{range .spec.containers[*]}{.image}{", "}{end}{end}'
kubectl get pod -A -o jsonpath="{.items[*].spec.containers[*].image}"
```
 ***
### Nodes operations:

```bash
# get nodes labes
kubectl get nodes --show-labels
kubectl get nodes -o wide

# add/delete label for node
kubectl label node <nodename> <labelname>=<value>
kubectl label node <nodename> <labelname>-

# describe node and check conditions field describes the status of all Running nodes: True if pressure
kubectl describe node <node_name>

# DRAIN to safely EVICT all of your pods from a node before you perform maintenance on the node (e.g. kernel upgrade, hardware maintenance)
kubectl drain <nodename>
kubectl drain <nodename> --ignore-daemonsets

# CORDON  mark node unschedulable for all pods and adds a taint node.kubernetes.io/unschedulable:NoSchedule to the node
kubectl cordon <node_name>
kubectl uncordon <node_name>
```
 ***
### kubectl configs:

 ```bash
# autocompletion 
source <(kubectl completion bash)
kubectl completion bash > ~/.kube/kubectl_autocompletion

# in ~/.bash_profile
if [ -f /usr/local/share/bash-completion/bash_completion ]; then
  . /usr/local/share/bash-completion/bash_completion
fi

# context - combination of a cluster and user credentials
kubectl config current-context
kubectl config get-clusters
kubectl config get-contexts

# switch context
kubectl config use-context <context_name>
# sets a cluster entry in kubeconfig
kubectl config set-cluster
# sets a context entry in kubeconfig
kubectl config set-context

# remove a context
kubectl --kubeconfig=config-demo config unset contexts.<name>
# remove a cluster
kubectl --kubeconfig=config-demo config unset clusters.<name>

kubectl config view
kubectl config view -o jsonpath='{.users[*].name}'
                 
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
kubectl get replicasets/rs
kubectl get deployments
kubectl get svc

# NAMESPACE = objects which partition a single K8s cluster into multiple virtual clusters
kubectl config view | grep namespace
kubectl get ns
kubectl config set-context --current --namespace=<namespace>
kubectl create ns <namespace>
```

 ***
### Deployment ops:

```bash
# set editor ftw
export KUBE_EDITOR=vim

# edit resource (bad-idea)
kubectl edit <resource_type>/<resource_name>

# interactive rollout
kubectl edit deployments/<deployment_name>

# scale rs to 1 for a deployment
kubectl -n <namespace> scale deployment <deployment_name> --replicas=1
```
 ***
### Services:

```bash
# get services
kubectl -n <namespace> get svc

# get service cluster-ip
kubectl get services/<service_name> -o go-template='{{(index .spec.clusterIP)}}'

# get external IP
kubectl -n <namespace> get services -o jsonpath='{.items[*].status.loadBalancer.ingress[0].ip}'

# get pod port containerPort
kubectl -n <namespace> get pod <pod_name> --template='{{(index (index .spec.containers 0).ports 0).containerPort}}{{"\n"}}'

# tunnels the traffic from a specified port at your local host machine to the specified port on the specified pod
kubectl port-forward pods/<pod_name> <port_no>:<port_no>
kubectl port-forward svc/nginx-service [LOCAL_HOST_PORT:]REMOTE_PORT
kubectl port-forward svc/nginx-service 80:8080 -n kube-public&
```
  ***

### 101 stuff

**Probes**:
- Liveness probe checks the container health and if it fails it restarts the container (kubelet uses liveness probes to know when to restart a container)
- Liveness probe fails => restart pod

- Readiness Probe check if a POD is ready to serve trafic (kubelet uses readiness probes to know when a container is ready to start accepting traffic. A Pod is considered ready when all of its containers are ready)
- Readiness probe fails => don't send any traffic to the pod

**Requests_and_limits**:
- CPU <k8s_CPU's> and memory <bytes> are each a resource type and form compute [resources](https://learnk8s.io/setting-cpu-memory-limits-requests), which can be requested, allocated, and consumed by containers.
- CPU resource units **limits** and **requests** are measured in CPU units.
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

**Customizing outptut**:
```bash
# just name
kubectl get po -o name -A

# custom columns .e.g NAME columns
kubectl get po -o=custom-columns=NAME:.metadata.name -A

# go templates are a powerful method to customize output however you want
kubectl get po -A -o go-template='{{range .items}} --> {{.metadata.name}} in namespace: {{.metadata.namespace}}{{"\n"}}{{end}}'
```
**Scheduling PODS on NODES**:

```bash

# this is the job of Kubernetes scheduler (who's responsible for the best Node for that Pod to run on), but is some situations it might be needed for us to contraint this process

# I want a certain POD to start on a specific NODE
kubectl label node <nodename> <labelname>=<value> # label node e.g label=chosen
kubectl edit deployment <deploymentname> # nodeSelector
spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: nodetype
                operator: In
                values:
                - chosen
```
**PODS and status**:

```bash
#check pods status
kubectl get po -A -o jsonpath='{.items[*].status}'

#A Pod's status field is a PodStatus object, which has a phase field.
kubectl get po -A -o jsonpath='{.items[*].status}' | jq . | grep -i phase
```

**Taint and Node affinity**:

* PODS get assigned to NODE using affinity feture and nodeSelector.
* Taints are used to repel Pods from specfic nodes - taint on a node allow only some pods (those with tolerations to the taint) to be scheduled on that node.

```bash
kubectl taint nodes -l LABEL=LABEL_VALUE KEY=VALUE:EFFECT

kubectl taint nodes <node_name> <taintKey>=<taintValue>:<taintEffect>
kubectl taint nodes host1 special=true:NoSchedule
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
