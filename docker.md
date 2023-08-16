## Cheatsheet collection

* [Home](index.md)
* [Ansible](ansible.md)
* [Git](git.md)
* [GCP](gcp.md)
* <ins>[Docker](docker.md)</ins>
* [Azure](azure.md)
* [Terraform](terraform.md)
* [Helm](helm.md)
* [ElasticSearch](elastic.md)
* [Kubernetes](k8s.md)
* [Istio](istio.md)
* [OIDC](openID.md)
* [PostgreSQL](postgres.md)

---

### Cleanup docker workspace

```bash
#stop all containers using their id
docker stop $(docker ps -q) 

# remove all stopped containers
docker rm -f $(docker ps -aq -f  status=exited) 

# remove all images
docker rmi -f $(docker images -a -q)

# remove all non-running containers
docker ps -a | grep Exited | awk '{print $1}' | xargs docker rm

# check images on your host machine
docker images
docker system info --format '{{ .Images}}'

# remove unused images
docker image prune -a

# remove stopped containers and delete dangling images
docker rm $(docker ps -aq -f status=exited) && docker image prune -a
```

### Go templates

* [go_templates](https://golang.org/pkg/text/template/)


```bash
## docker list containers using Pretty-print containers using a Go template instead of using | and awk
docker ps --format "{{.Names}} with {{.Status}}"
docker ps --format '{{ .ID }}'

# as a table
docker ps --format "table {{.Names}} {{.Status}}"

### check mounts volumes and binds
docker inspect -f '{{ .Name }}{{ printf "\n" }}{{ range .Mounts }}{{ printf "\n\t" }}{{ .Type }} {{ if eq .Type "bind" }}{{ .Source }}{{ end }}{{ .Name }} => {{ .Destination }}{{ end }}{{ printf "\n" }}' <INSERT_IMAGE_ID>

# list running containers names
docker ps --filter status=running --format '{{.Names}} {{.Status}}'

# inspect ENV variables
docker inspect --format '{{ .Config.Env}}' <IMAGE>/<CONTAINER>
docker inspect --format='{{range .Config.Env}}{{println .}}{{end}}' <CONTAINER>

# inspect volumes and their type
docker inspect -f '{{ .Mounts }}' <CONTAINER>

# inspect mounted volumes for a container
docker inspect --format='{{range .Mounts}}{{println .Source}}{{end}}' <CONTAINER>

# list all mounts for all running containers
docker ps --no-trunc --format "{{.ID}}\t{{.Names}}\t{{.Mounts}}"

# inspect env and entrypoint/cmd
docker inspect -f '{{ .Config.Env}} {{ .Config.Entrypoint}} {{ .Config.Cmd}}' <CONTAINER>

# inspect docker network and attached containers | jq .
docker inspect network bridge --format "{{json .Containers }}" | python -m json.tool

# nice table
docker ps --format "table {{.Names}} {{.Status}}"
```

### Docker images/registry 
```bash

# save image as tarball
docker save -o tarfile.tar <IMAGE>
docker save <IMAGE> | gzip > <IMAGE>.tar.gz

# load image
docker load < <IMAGE>.tar.gz

# save and transfer image to another server
docker save <IMAGE> | bzip2 | ssh hostname.fqdn docker load 
docker save dejanualex/exporter | bzip2 | ssh hostname.fqdn docker load 

# list images
curl -k -X GET https://<USER>:<PASSWORD>@<REGISTRY>/v2/_catalog | python -m json.tool
curl -k -X GET https://<USER>:<PASSWORD>@<REGISTRY>/v2/_catalog | jq .
```
### Manage Docker
```bash

# docker system info
docker system info --format '{{ .ServerVersion }} {{ .Driver }}'


# Usage:  docker system COMMAND

Commands:
  df          Show docker disk usage
  events      Get real time events from the server
  info        Display system-wide information
  prune       Remove unused data
```

### Docker containers and networking
```bash
# get the container PID
docker inspect --format '{{ .State.Pid }}' <CONTAINER_ID>

# shim abstract low-level details of the container runtime
# containerd-shim process in between containerd and bash
for j in $(for i in $(ps -C containerd-shim | awk 'FNR>1 {print $1}');do pgrep -P $i;done);do ps -p $j -o comm=;done

## list network interfaces
# default network driver is bridge
# overlay is the default network driver for swarm, it connects multiple Docker daemons together and enable swarm services to communicate with each other
docker network ls

# list containers in a network
docker network inspect bridge --format "{{json .Containers }}"

```
### Docker logging
```bash
# /etc/docker/daemon.json
{
  "log-driver": "journald"
}

docker info --format '{{.LoggingDriver}}'
docker info -f '{{ .DockerRootDir}}'
# check the security module
docker system info --format '{{ .SecurityOptions}}'

docker inspect --format='{{.LogPath}}' <CONTAINER_ID>

# follow logs starting from the last 10 lines onwards
docker logs -f --tail 10 <CONTAINER_ID>
docker logs --since=1h <CONTAINER_ID>
docker logs <CONTAINER_ID> --since 10m --follow

# redirect stdout and stderr to a file
docker logs -f <CONTAINER_ID> > &> container.log
docker logs <CONTAINER_ID> > container.log 2>&1

# get container stats
docker stats <CONTAINER_ID>

# check container states e.g: OOMKilled, Dead (very usefull whend debugging)
docker container inspect prometheus_normal | jq .[].State
```
### Docker registry
```bash

## search images
curl -k -XGET https://<USER>:<PASSWORD>@<REGISTRY>/v2/_catalog
docker search <REGISTRY>/<IMAGE>

# endpoint for docker registry
/v2/_catalog 
/v2/<IMAGE>/tags/list

# search dockerhub for images
docker search --format "{{.Name}}: {{.StarCount}}: {{.IsOfficial}}" httpd
docker search --format "{{.Name}}: {{.StarCount}}: {{.IsOfficial}}" nginx

# manifest schem https://docs.docker.com/registry/spec/manifest-v2-2/
# manifest command interacts solely with a Docker registry.
# Docker manifests describe an image’s layers and the architectures it supports
docker manifest inspect maven
```
### Docker swarm stuff:
```bash
docker stack ls
docker stack service <stack_name>
docker service ps <service_name>
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
