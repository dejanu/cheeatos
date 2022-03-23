## Cheatsheet collection

* [Home](#)
* [Ansible](ansible.md)
* [Git](git.md)
* [GCP](index.md)
* [Docker](docker.md)
* [Azure](azure.md)
* [Terraform](terraform.md)
* [Helm](helm.md)
* [ElasticSearch](elastic.md)
* <ins>[Istio](istion.md)</ins>

### General features

* observability(telemetry) - metrics and access logs
* traffic management -> mesh network for servicesv -> routing roules, circuit breakers also great for deployments
* security: mutuals TLS, req auth. E.g Mutal TLS = Secure communication between services inside the mesh between PODs with the Istio sidecar injected and also from mesh-external services accessing the mesh through an ingress gateway.


* Services give SPAN (piece of info of each server/servce provide) to trace a request - TELEMETRY

* Istio injects a ENVOY proxy side-car container to the pods to intercept traffic from pods, this behavior is enabled using label `istio-injection=enabled` on the namespace level and `sidecar.istio.io/inject=true` on the pod level.
* ENVOY proxy is deployed as a sidecar and mediates all inbound/outbound traffic for services in the mesh and enabls load balancing, circuit brakers, fault injection

* Mesh traffic out of the box is mutals TLS (both client and server side)
* `istio-init` This init container is used to setup the iptables rules so that inbound/outbound traffic will go through the sidecar proxy

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

### traffic routing  to kubernetes services with Istio (Ingress and Egress traffic)

* Istio routing rules provide fine-grained control over how to route traffic based on host, port, headers, uri, method, source labels and control the distribution of traffic.

```bash
# kind: Rule
# virtualservice.networking.istio.io/name-route created
# destinationrule.networking.istio.io/name-destination-rule created

- VirtualService - how to route the traffic where to( which service) /foo or /bar
- DestinationRule - what version of service /foo v1 or v2 - what happens to traffic for a destination defined in a virtual service
```

* Istio has its own Ingress Gateway (Gateway controller which runs a standalone envoy proxy)
* Igress (traffic into the mesh) and Egress (traffic out of the mesh)
* Istio by default has an outbound traffic policy which is "passthrough" by default

* A Gateway describes a load balancer operating at the edge of the mesh receiving incoming or outgoing HTTP/TCP connections. The specification describes the ports to be expose, type of protocol, configuration for the load balancer, etc.

### Service level metrics

* Istio's envoy proxies generate the distributed trace SPAN for the services.

* Istion generates detail metrics telemetry for all traffic within the mesh (proxy-level, service-level)
* You should enable prometheus integration
* the envoy proxy sidecar automatically generates trace spans on behalf of the application it proxies - Distributed trace spans.

### Best practices

* Set defaults routes for every service
* Split large virtual services and destination rules
* Apply destionation rules to first
* https://istio.io/latest/docs/ops/best-practices/security/
