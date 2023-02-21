## DevOps and SRE workshop ![gke](https://github.com/dejanu/cheetcity/blob/gh-pages/src/gke.svg?raw=true)

* [Home](index.md)

### Docker

* list containers using Pretty-print containers with a Go template instead of using `|` and `awk`:

 docker ps --format "{{.Names}} with {{.Status}}"

* how can I remove unused image/dangling images and stopped containers? (filter comes in handy). Remove stopped containers and remove unused images not just dangling ones:

docker rm $(docker ps -aq -f status=exited) && docker image prune -a

* how can you list running containers using filters?

docker ps --filter status=running --format '{{.Names}} {{.Status}}'

* how can you quickly asses the status of docker daemon, docker disk usage, and docker containers?

docker system info
docker system df

* what is the default network driver for docker? how can you list docker networks?

docker network ls

```bash
# default network driver is bridge
# overlay is the default network driver for swarm, it connects multiple Docker daemons together and enable swarm services to communicate with each other
```

* how can I check the logs of a container (in the last 60 minutes)?

docker logs --since 60m <CONTAINER>
docker logs --since=1h <CONTAINER_ID>

### Kubernetes

* how can I get the logs of a specific container in a pod?
```bash
kubectl logs -f <POD_NAME> -c <CONTAINER_NAME>
```

```bash
                    ___ _____
                   /\ (_)    \
                  /  \      (_,
                 _)  _\   _    \
                /   (_)\_( )____\
                \_     /    _  _/
                  ) /\/  _ (o)(
                  \ \_) (o)   /
                   \/________/         @systematic
```