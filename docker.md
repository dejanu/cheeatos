## Cheatsheet collection

* [Home](#)
* [Ansible](ansible.md)
* [Git](git.md)
* [GCP](index.md)
* <ins>[Docker](docker.md)</ins>

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
```

### Go templates

https://golang.org/pkg/text/template/

```bash

# check mounts
docker ps --format '{{ .ID }}' | xargs -I {} docker inspect -f '{{ .Name }}{{ printf "\n" }}{{ range .Mounts }}{{ printf "\n\t" }}{{ .Type }} {{ if eq .Type "bind" }}{{ .Source }}{{ end }}{{ .Name }} => {{ .Destination }}{{ end }}{{ printf "\n" }}' {}

docker ps --no-trunc --format "{{.ID}}\t{{.Names}}\t{{.Mounts}}"

docker inspect -f '{{ .Mounts }}' <CONTAINER_ID>
```

### Docker registry search
```bash

curl -k -X GET https://<USER>:<PASSWORD>@<REGISTRY>/v2/_catalog | python -m json.tool
```
