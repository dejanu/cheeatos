## Cheatsheet collection

* [Home](#)
* [Ansible](ansible.md)
* [Git](git.md)
* [GCP](index.md)
* <ins>[Docker](docker.md)</ins>
* [Azure](azure.md)
* [Terraform](terraform.md)

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

# remove unused images
docker image prune -a

# remove stopped containers and delete dangling images
docker rm $(docker ps -aq -f status=exited)&& docker image prune -a
```

### Go templates

https://golang.org/pkg/text/template/


```bash


# check mounts
docker ps --format '{{ .ID }}' | xargs -I {} docker inspect -f '{{ .Name }}{{ printf "\n" }}{{ range .Mounts }}{{ printf "\n\t" }}{{ .Type }} {{ if eq .Type "bind" }}{{ .Source }}{{ end }}{{ .Name }} => {{ .Destination }}{{ end }}{{ printf "\n" }}' {}

# list running containers names
docker ps --filter status=running --format '{{.Names}} {{.Status}}'

# inspect ENV variables
docker inspect --format '{{ .Config.Env}}' <IMAGE>/<CONTAINER>
docker inspect --format='{{range .Config.Env}}{{println .}}{{end}}'

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

# transfer image to another server
docker save dejanualex/exporter | bzip2 | ssh hostname docker load 

# list images
curl -k -X GET https://<USER>:<PASSWORD>@<REGISTRY>/v2/_catalog | python -m json.tool
curl -k -X GET https://<USER>:<PASSWORD>@<REGISTRY>/v2/_catalog | jq .
```


### Docker logging
```bash
# /etc/docker/daemon.json
{
  "log-driver": "journald"
}

docker info --format '{{.LoggingDriver}}'
docker inspect --format='{{.LogPath}}' <CONTAINER_ID>

# follow logs starting from the last 10 lines onwards
docker logs -f --tail 10 <CONTAINER_ID>
docker logs --since=1h <CONTAINER_ID>

# redirect stdout and stderr to a file
docker logs -f <CONTAINER_ID> > &> container.log
docker logs <CONTAINER_ID> > container.log 2>&1

# get container stats
docker stats <CONTAINER_ID>
```

