## Cheatsheet collection

* [Home](https://dejanu.github.io/)
* [Ansible](ansible.md)
* [Git](git.md)
* [GCP](index.md)
* [Docker](docker.md)

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
