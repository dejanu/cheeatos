## Cheatsheet collection

* [Home](index.md)

## DevOps and SRE workshop ![gke](https://github.com/dejanu/cheetcity/blob/gh-pages/src/gke.svg?raw=true)

### Docker

* list containers using Pretty-print containers with a Go template instead of using `|` and `awk`
```bash
 docker ps --format "{{.Names}} with {{.Status}}"
```
* how can I remove unused image/dangling images and stopped containers? (filter comes in handy)
```bash
# remove stopped containers and emove unused images not just dangling ones
docker rm $(docker ps -aq -f status=exited) && docker image prune -a
```
* how can you list running containers using filters?
```bash
docker ps --filter status=running --format '{{.Names}} {{.Status}}'
```
* how can you quickly asses the status of docker daemon, docker disk usage, and docker containers?
```bash
docker system info
docker system df
```
* what is the default network driver for docker? how can you list docker networks?
```bash
# default network driver is bridge
# overlay is the default network driver for swarm, it connects multiple Docker daemons together and enable swarm services to communicate with each other
docker network ls
```
* how can I check the logs of a container (in the last 60 minutes)?
```bash
docker logs --since 60m <CONTAINER>
docker logs --since=1h <CONTAINER_ID>


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