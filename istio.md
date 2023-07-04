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
* [Kubernetes](k8s.md)
* <ins>[Istio](istio.md)</ins>
* [OIDC](openID.md)
* [PostgreSQL](postgres.md)

---

### General features

* Getting external traffic into your cluster can be achieved using the Service k8s object or Ingress
* Ingress is probably the most powerful way to expose your services, but can also be the most complicated:
  * There are many types of Ingress controllers, from the Google Cloud Load Balancer, Nginx, Contour, ISTIO, and more. 
  * There are also plugins for Ingress controllers, like the cert-manager, that can automatically provision SSL certificates for your services.

* To expose traffic we need a Ingress Controller (e.g. ISTIO) + Ingress rules
* Service mesh is an abstraction which consists of a network of **envoys** and the **istiod** - one envoy in the ingress gateway and one envoy in each pod (the istio proxy)
* If you expose another service as type LoadBalancer and obtains a separate ip, then requesting that endpoint would still be regarded as ingress traffic but not manager by the Istio service mesh.

* The ingress gateway is just the first workload part of the service mesh to intercept and process the request.
* Istio uses gateway instead of ingress:
```bash
kubectl get ingresses.networking.k8s.io -A  # no resources found
kubectl get gateways.networking.istio.io -A # is a CRD used by Istio Service mesh
```
* Istio envoy proxy that acts as a LB
* Gateway describes a load balancer operating at the edge of the mesh receiving incoming or outgoing HTTP/TCP connections.

* Flow `external request -> svc (loadbalancer type) -> istio ingress pod -> istio gateway -> istio virtualservice -> application pod`
* Side note: istio gateway and virtual service are just intrumentation to the ingress pod

* Istio injects a ENVOY proxy side-car container to the pods to intercept traffic from pods, this behavior is enabled using label `istio-injection=enabled` on the namespace level and `sidecar.istio.io/inject=true` on the pod level.

* ENVOY proxy is deployed as a sidecar and mediates all inbound/outbound traffic for services in the mesh and enabls load balancing, circuit brakers, fault injection

* Mesh traffic out of the box is mutals TLS (both client and server side). Mutal TLS = Secure communication between services inside the mesh between PODs with the Istio sidecar injected and also from mesh-external services accessing the mesh through an Ingress Gateway.

* `istio-init` This init container is used to setup the iptables rules so that inbound/outbound traffic will go through the sidecar proxy

* Spi-up a pod without Istio: `kubectl run mybusyboxcurl --labels="sidecar.istio.io/inject=false" --image yauritux/busybox-curl -it -- sh`

* Disable the sidecar container, updating the label:
```yml
 podLabels:
      sidecar.istio.io/inject: "false"
```

### Sidecard

* Set of proxies running along the services
* Sidecar describes the configuration of the sidecar proxy that mediates inbound/outbound communication to the workload instance it is attached to.
* Sidecar injection is adding the configuration of additional containers to the pod template.

### Architecture

* DATA PLANE (set of proxies/sidecars e.g: [Envoy]) + CONTROL PLANE (it provides certs, config the proxies to route - istiod)
* you must have to have a k8s service !!!
* CONTROL PLANE has 3 components:

```bash
1) Pilot - service discovery and traffic management with Envoy
2) Galley - config validation - only component that interact with with k8s
3) Citadel - cert management: istio has its own builtin CA
```

### Traffic routing  to kubernetes services with Istio (Ingress and Egress traffic)

* Istio routing rules provide fine-grained control over how to route traffic based on host, port, headers, uri, method, source labels and control the distribution of traffic.

* Istio resources:
    * VirtualService - how to route the traffic where to (which service) /foo or /bar 
    * DestinationRule - policies that apply to traffic after it has been routed through VeirutaService. What version of service /foo v1 or v2 - what happens to traffic for a destination defined in a virtual service.
    * Gateway - describes the Load Balancer (ports + Server Name Indicator)

```bash
# get all Istio resource from default namespace
kubectl get dr,gw,vs -n default

# kind: Rule
destinationrule.networking.istio.io/frontend created
gateway.networking.istio.io/frontend-gateway created
virtualservice.networking.istio.io/frontend-vs created
```

* Istio has its own Ingress Gateway (Gateway controller which runs a standalone envoy proxy)
* Igress (traffic into the mesh) and Egress (traffic out of the mesh)
* Istio by default has an outbound traffic policy which is "passthrough" by default

* A Gateway describes a load balancer operating at the edge of the mesh receiving incoming or outgoing HTTP/TCP connections. The specification describes the ports to be expose, type of protocol, configuration for the load balancer, etc.

### Service level metrics

* Istio's envoy proxies generate the distributed trace SPAN for the services.
* Istion generates detail metrics telemetry for all traffic within the mesh (proxy-level, service-level)
* You should enable prometheus integration
* Ehe envoy proxy sidecar automatically generates trace spans on behalf of the application it proxies - Distributed trace spans.

### Best practices

* Set defaults routes for every service
* Split large virtual services and destination rules
* Apply destionation rules to first
* https://istio.io/latest/docs/ops/best-practices/security/

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
