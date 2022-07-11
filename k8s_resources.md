

<ins>[Back](k8s.md)<ins>
---

| Name 	               | ShortName 	                |
|----------------------|----------------------------|
|bindings              | Binding    	            |
|configmaps            |  	cm                      |
|endpoints             |  	ep                      |
|events                |  	ev                      |
|namespaces            |  	ns                      |
|persistentvolumeclaims|  	ev                      |
|persistentvolumes     |  	pv                      |
|pods                  |  	po                      |
|replicationcontrollers|ReplicationController       |
|services              |  	svc                     |
|secrets               |Secret                      |
|daemonsets            |    ds                      |
|deployments           |  	deploy                  |
|replicasets           |  	rs                      |
|statefulsets          |  	srs                     |



### Notes:

* [getting_started](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#-strong-getting-started-strong-)

* explain specs: `kubectl explain pod.spec.containers`

* check them: `kubectl api-resources`

* daemonset = 1 replica on each worker nodes aka 1 special pod on all workers e.g. ingress_controller exposed via a NodePort
* job = sort of task, k8s will schedule a job and it will run it once and it will not be rescheduled - 1 time, when u apply it it gets executed and that's it aka init container
* cronjob = a job that runs on a schedule
* ConfigMap = key/value pairs of configuration data that can be accessed by pods
* Secret = key/value pairs of sensitive data that can be accessed by pods (encoded in base64)) so `describe` will show opaque data 

---



```bash
# SECRETS example scenario

kubectl create secret generic <secret_name> --from-literal=MYSQL_ROOT_PASSWORD=test
kubectl get secrets <secret_name> -o yaml > sqlsecret.yml

kubectl exec -it po/<pod_name> -- bash
# test the secret 
mysql -u root -p
```

![alt text](https://github.com/dejanu/cheetcity/blob/gh-pages/src/k8s_objects_me.png?raw=true)