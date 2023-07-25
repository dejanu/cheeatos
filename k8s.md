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

### Explain specs:

```bash
# understand container specs
kubectl explain pod.spec.containers.resources

# understand node specs
kubectl explain node.spec
```
### Control plane checks:

```bash
# get control plane status
kubectl get cs
kubectl get componentstatuses

# get version k8s
kubectl version --short

# API endpoints for health checks without formatting
kubectl get --raw '/livez?verbose'
kubectl get --raw '/readyz?verbose'
kubectl get --raw "/healthz?verbose"    

# kubectl top nodes
kubectl get --raw "/apis/metrics.k8s.io/v1beta1/nodes"

# ingress endpoint
kubectl get --raw "/apis/networking.k8s.io/v1/ingresses/"

# check events
kubectl get events --sort-by='.metadata.creationTimestamp' -A
kubectl get events --field-selector type!=Normal -A
kubectl get events --field-selector type=Warning -A

# check events for specific pod
kubectl get events --field-selector involvedObject.name=<pod_name> -A
```

### Debugging pods:

```bash
# flags: allowed formats are: custom-columns,custom-columns-file,go-template,go-template-file,json,jsonpath,jsonpath-file,name,template,templatefile,wide,yaml
kubectl get pods -A (--all-namespaces )
kubectl get pods --show-labels
kubectl get pods -w (--watch)
kubectl get pods -o json

# list events
kubectl describe pod/<pod_name> -n <namespace>

# get pod restart counts
kubectl get po -A --sort-by='.status.containerStatuses[0].restartCount'
kubectl get pods -A --sort-by=.status.containerStatuses[0].restartCount!=0

# running naked debug pod (svc in the same namespace are resolvable by DNS)
kubectl run -ti kali --image=kalilinux/kali-rolling
kubectl run -n <namespace> mybusyboxcurl --image yauritux/busybox-curl -it -- sh
kubectl delete -n default pod kali

# spin up a shell inside the pod
kubectl exec -n <namespace> -it <pod_name> -- bash

# execute cmd:e.g. curl from one pod to another
kubectl exec -t -n <namespace> <pod_name> -- curl -I http://<another_pod_ip>:3030/metrics

# get pod logs from all namespaces
for n in $(kubectl get ns | awk 'FNR>1 {print $1}');do kubectl get pods -n $n;done

# logs for for a specific RESOURCE: deployment is specified and that deployment has multiple pods such as a ReplicaSet
# then only one of the pods logs will be returned
kubectl -n <namespace> logs deployment/<deploymet_name> --all-containers=true --since 10m

# get logs (no_lines) from a specific POD
kubectl -n <namespace> logs <pod_name> --tail 200 --timestamps=true

# check pod running images
kubectl get pods -A -o=jsonpath='{range .items[*]}{"\n"}{.metadata.name}{":\t"}{range .spec.containers[*]}{.image}{", "}{end}{end}'
kubectl get pod -A -o jsonpath="{.items[*].spec.containers[*].image}"

# delete po with label=fluent-bit
kubectl -n logging delete po -l app.kubernetes.io/instance=fluent-bit

# delete po with label
kubectl -n monitoring delete po -l app=prometheus-node-exporter

# get pods with label app=flux from all namespaces
kubectl get pod -A -l app=flux -oname
kubectl -n monitoring get po -l "app=kiali" -oname

# force delete pod and patch the finalizers
kubectl -n redis delete pods <pod> --grace-period=0 --force
kubectl -n redis patch pod <pod> -p '{"metadata":{"finalizers":null}}'
```

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

### kubectl configs:

* Kubernetes [context like a pro](https://community.ops.io/dejanualex/kubectl-context-like-a-pro-2692)

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
kubectl config delete-context <context_name>
# remove a cluster
kubectl --kubeconfig=config-demo config unset clusters.<name>

kubectl config view
kubectl config view -o jsonpath='{.users[*].name}'
                 
# Sets a user entry in kubeconfig
kubectl config set-credentials

# Sets the current-context in a kubeconfig file
kubectl config use-context
```
 
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

### Scale down daemonset:

```bash
# to scale down a DaemonSet one can use the following workaround: 
# basically by adding a temporary nodeSelector that matches no nodes, making the DaemonSet pods un-schedulable on any nodes:

kubectl -n <namespace> patch daemonset <name-of-daemon-set> -p '{"spec": {"template": {"spec": {"nodeSelector": {"non-existing": "true"}}}}}'

# remove the NodeSelector to scale up the daemonset
kubectl -n <namespace> patch daemonset <name-of-daemon-set> --type json -p='[{"op": "remove", "path": "/spec/template/spec/nodeSelector/non-existing"}]
```

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

### CRD:

```bash
# Resource = endpoint in k8s K8s API
# Custom Resource = extension of the Kubernetes API that is not necessarily available in a default Kubernetes installation
kubectl get crd -n <namespace>
kubect get <customresource_name> -n <namespace>
kubectl describe  <customresource_type> <customresource_name> -n <namespace>
```

### Customizing outptut:

```bash
# just name
kubectl get po -o name -A

# custom columns .e.g NAME columns
kubectl get po -o=custom-columns=NAME:.metadata.name -A

# go templates are a powerful method to customize output however you want
kubectl get po -A -o go-template='{{range .items}} --> {{.metadata.name}} in namespace: {{.metadata.namespace}}{{"\n"}}{{end}}'
```


### Scheduling PODS on NODES:

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


### PODS and status:

```bash
#check pods status
kubectl get po -A -o jsonpath='{.items[*].status}'

#A Pod's status field is a PodStatus object, which has a phase field.
kubectl get po -A -o jsonpath='{.items[*].status}' | jq . | grep -i phase
```


### Taint and Node affinity:

```bash

# PODS get assigned to NODE using affinity feture and nodeSelector.
# Taints are used to repel Pods from specfic nodes - taint on a node allow only some pods (those with tolerations to the taint) to be scheduled on that node.
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
