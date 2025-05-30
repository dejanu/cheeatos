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
* <ins>[Kubernetes](k8s.md)<ins> -> [raw k8s](https://raw.githubusercontent.com/dejanu/sretoolkit/master/k8s_stuff/kubestuff)
* [Istio](istio.md)
* [OIDC](openID.md)
* [PostgreSQL](postgres.md)
* [GitHub Copilot](copilot.md)

---

### Explain specs:

```bash
kubectl explain <resource>.<key>

# understand container specs
kubectl explain pod.spec.containers
kubectl explain pod.spec.containers.resources

# understand node specs
kubectl explain node.spec
```
### Control plane checks:

```bash
kubectl cluster-info

# get control plane status
kubectl get cs
kubectl get componentstatuses

# get version k8s
kubectl version --short

# check available API resources: cm,deployments,ds,ingress,ns,pods,rc,rs,secrets,svc
kubectl api-resources

# check api services in the cluster: v1.certificates.k8s.io, v1.networking.io
kubectl get apiservices

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

### TLS access:

```bash
# get cert client-certificate-data info from kubeconfig
export client=$(grep client-cert $HOME/.kube/config |cut -d" " -f 6)

# get client-key-data info from kubeconfig
export key=$(grep client-key-data $HOME/.kube/config |cut -d " " -f 6)

# get client-key-data info from kubeconfig
export auth=$(grep certificate-authority-data $HOME/.kube/config |cut -d " " -f 6)

# encode the keys
echo $client | base64 -d - > ./client.pem
echo $key | base64 -d - > ./client-key.pem
echo $auth | base64 -d - > ./ca.pem

# get the API url: API_url
kubectl config view |grep server
# extract the API server url
API_SERVER=$(kubectl config view --minify -o jsonpath='{.clusters[].cluster.server}')

## With the appropriate TLS keys you could run curl
curl --cert ./client.pem --key ./client-key.pem --cacert ./ca.pem https://<API_URL>/api/v1/pods

# check what a user (e.g. bob) can do in a certain namespace:
kubectl auth can-i create deployments --as bob --namespace developer 

# create token secret
export token=$(kubectl create token default)

# check the token pass -k for insecure and skip using a cert
curl https://<API_URL>/apis --header "Authorization: Bearer $token" -k

# get user
kubectl config get-users

# extract the token for the desired user (replace <user_name> with the actual name of the user
kubectl config view --raw -o jsonpath='{.users[?(@.name=="<cluster_username>")].user.token}'

# call API server authentication endpoint
kubectl get --raw $API_SERVER/apis/authentication.k8s.io/v1/
kubectl get --raw https://phi-synergy-76435e55.hcp.westeurope.azmk8s.io:443/apis/authentication.k8s.io/v1/

# extract the cert and the key
kubectl config view --raw -o jsonpath='{.users[?(@.name=="<cluster_username>")].user.client-certificate-data}' | base64 -d
kubectl config view --raw -o jsonpath='{.users[?(@.name=="<cluster_username>")].user.client-key-data}' | base64 -d

# query the authorization layer using a token and user
kubectl auth can-i --list --token=$TOKEN --as=$USER
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
kubectl get po -A --sort-by=.status.containerStatuses[0].restartCount!=0

# sort pod by IP
kubectl get po -A --sort-by=.status.podIP -owide

# running naked debug pod (svc in the same namespace are resolvable by DNS)
kubectl run -ti kali --image=kalilinux/kali-rolling
kubectl run -n <namespace> mybusyboxcurl --image yauritux/busybox-curl -it -- sh
kubectl delete -n default pod kali

# spin up a shell inside the pod
kubectl exec -n <namespace> -it <pod_name> -- bash

# execute cmd:e.g. curl from one pod to another
kubectl exec -t -n <namespace> <pod_name> -- curl -I http://<another_pod_ip>:3030/metrics

# get pods from all namespaces
for n in $(kubectl get ns | awk 'FNR>1 {print $1}');do kubectl get pods -n $n;done

# Kubernetes will capture anything written to standard output and standard error as a log message
kubectl -n <namespace> logs <pod_name> --all-containers
kubectl -n <namespace> logs <pod_name> -c <container> --since 10m

# logs for for a specific RESOURCE: deployment is specified and that deployment has multiple pods such as a ReplicaSet
kubectl -n <namespace> logs deployment/<deployment_name> --all-containers=true --since 10m  --timestamps=true

# get logs (no_lines) from a specific POD
kubectl -n <namespace> logs <pod_name> --tail 200 --timestamps=true

# check pod running images
kubectl get pods -A -o=jsonpath='{range .items[*]}{"\n"}{.metadata.name}{":\t"}{range .spec.containers[*]}{.image}{", "}{end}{end}'
kubectl get pod -A -o jsonpath="{.items[*].spec.containers[*].image}"


# get container running in a SINGLE POD
kubectl -n <namespace> get pod <pod_name>  -o jsonpath="{.spec.containers[*].name}"

# get container running in a NAMESPACE
kubectl -n <namespace>  get pod -o jsonpath="{.items[*].spec.containers[*].name}"

# get all pods with a certain label e.g. run
kubectl get pods -L run

# get pod all pods with a certain value for a label e.g. run=ghost
kubectl get pods -l run=ghost
kubectl get po -l "app=kiali" -oname

# delete po with label=fluent-bit
kubectl -n logging delete po -l app.kubernetes.io/instance=fluent-bit

# list all pods with label colour=orange or red or yellow
kubectl get pods --selector 'colour in (orange,red,yellow)' --show-labels

# delete po with label=fluent-bit
kubectl -n logging delete po -l app.kubernetes.io/instance=fluent-bit

# ovwerwrite labels
kubectl label po <pod_name> sidecar.istio.io/inject=false --overwrite

# get pods with label app=flux from all namespaces
kubectl get pod -A -l app=flux -oname
kubectl -n monitoring get po -l "app=kiali" -oname

# force delete pod and patch the finalizers
kubectl -n redis delete pods <pod> --grace-period=0 --force
kubectl -n redis patch pod <pod> -p '{"metadata":{"finalizers":null}}'

# check pods status
kubectl get po -A -o jsonpath='{.items[*].status}'

# a Pod's status field is a PodStatus object, which has a phase field.
kubectl get po -A -o jsonpath='{.items[*].status}' | jq . | grep -i phase
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
kubectl drain <nodename> --ignore-daemonsets

# CORDON  mark node unschedulable for all pods and adds a taint node.kubernetes.io/unschedulable:NoSchedule to the node
kubectl cordon <node_name>
kubectl uncordon <node_name>
```

### kubectl configs:

* Kubernetes [context like a pro](https://community.ops.io/dejanualex/kubectl-context-like-a-pro-2692)

```bash
# set editor ftw
export KUBE_EDITOR=vim

# list completions
kubectl completion -h

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
                 
# sets a user entry in kubeconfig
kubectl config set-credentials

# set default namespace
kubectl config set-context --current --namespace=<namespace>
kubectl config set-context $(kubectl config current-context) --namespace=<namespace>

# merging two config files(one for each cluster) 
# create backup for current config
cp ~/.kube/config ~/.kube/config.bak

KUBECONFIG=~/.kube/config:/path/to/new/config kubectl config view --flatten > ~/intermediate_config
mv ~/intermediate_config ~/.kube/config
```
 
### Get k8s objects:

```bash
# replicasets/nodes/pods/services/deployments/daemonsets/statefulsets/cronjobs
kubectl get po,deploy,svc,rs


# NAMESPACE = object which partitions a single K8s cluster into multiple virtual clusters
kubectl get ns
kubectl config set-context --current --namespace=<namespace>
kubectl create ns <namespace>
```
### Check limist and requests:

```bash
kubectl -n <namespace> get pod <pod_name> -o jsonpath='{.spec.containers[*].resources.limits}'
kubectl -n <namespace> get pod <pod_name> -o jsonpath='{.spec.containers[*].resources.requests}'
```

### Get names and customize output:
```bash
kubectl get po -o=custom-columns=NAME:.metadata.name -A
# Go templates are a powerful method to customize output however you want
kubectl get po -A -o go-template='{{range .items}} --> {{.metadata.name}} in namespace: {{.metadata.namespace}}{{"\n"}}{{end}}'
kubectl get po -o name -A
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

# get external IP of a LB service
kubectl -n <namespace> get services -o jsonpath='{.items[*].status.loadBalancer.ingress[0].ip}'

# get pod port containerPort
kubectl -n <namespace> get pod <pod_name> --template='{{(index (index .spec.containers 0).ports 0).containerPort}}{{"\n"}}'

# tunnels the traffic from a specified port at your local host machine to the specified port on the specified pod
kubectl port-forward pods/<pod_name> <port_no>:<port_no>
kubectl port-forward svc/nginx-service [LOCAL_HOST_PORT:]REMOTE_PORT
kubectl port-forward svc/nginx-service 80:8080 -n kube-public&

# creates a local service to access a ClusterIP, usefull for troubleshooting and provides quick way to check your service
kubectl proxy
```

### CRD:

```bash
# Resource = endpoint in K8s API
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

# Nodeselector: schedule a Pod on a node based on a label
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

### Taint and Node affinity:

# PODS get assigned to NODE using affinity feature and nodeSelector...

# Taint are similar to labels but they have an effect associated with them.
# Taints are used to repel Pods from specfic nodes - taint on a node allow only some pods (those with tolerations to the taint) to be scheduled on that node.

kubectl taint nodes <node_name> <taintKey>=<taintValue>:<taintEffect>
kubectl taint nodes host1 special=true:NoSchedule

# Taint effects: NoSchedule, PreferNoSchedule and NoExecute
kubectl taint nodes worker bubba=value:PreferNoSchedule
# NoSchedule - prevents scheduling of new pods
# PreferNoSchedule - prevents scheduling of new pods unless no other nodes are available
# NoExecute - evicts all exitind pods that do not tolerate the taint
```


### Admin stuff:

```bash
# using kubeadm you can create a minimum viable Kubernetes cluster
# control plane nodes init 
kubeadm init
# worker node init
kubeadm join

# create the network: e.g. Weave network
kubectl create -f https://git.io/weave-kube

# check weave status
kubectl -n kube-system get po -l "name=weave-net" -owide
for p in $(kubectl -n kube-system get po -l "name=weave-net" -oname);do echo $p;kubectl -n kube-system exec $p  -- /home/weave/weave --local status connections ;done 
```
---

### Links:
* [Intro to kubectl](https://kubectl.docs.kubernetes.io/guides/introduction/kubectl/)
* [kubectl cheatsheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
* [Kubeconfig fields](https://kubernetes.io/docs/reference/config-api/kubeconfig.v1/)
* [Getting started](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#-strong-getting-started-strong-)
* [Kubectl book](https://kubectl.docs.kubernetes.io/) and API [mgmt](https://kubernetes.io/docs/concepts/overview/working-with-objects/object-management/)
* [killercoda](https://killercoda.com/)
* [play with k8s](https://labs.play-with-k8s.com/)
* [Nice repo for CKA](https://github.com/walidshaari/Kubernetes-Certified-Administrator)

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
