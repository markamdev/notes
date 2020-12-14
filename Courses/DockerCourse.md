# Docker learning course notes

This document is a notebook for Docker course by Bret Fisher on Udemy:

* [Docker Mastery: With Kubernetes +Swarm](https://udemy.com/course/docker-mastery/learn/)

## Base info

Docker Mastery course (from Udemy)
Slack: https://chat.bretfisher.com/
Docker store: store.docker.com
Docker in browser: play-with-docker.com
Installation on ubuntu: https://docs.docker.com/engine/install/ubuntu/
Info page (shell settings): www.bretfisher.com/shell/

## Command notes

Short Docker command cheat sheet

* docker container ls <- list of running containers
* docker container ps <- as above
* docker container top XXX <- processes launched inside running container XXX
* docker container run
* * --detach / -d <- detach console from container
* * --name XYZ <- set name for this container; warning: it's persistent (till container removal)
* * --env ABC=xyz / -e ABC=xyz <- set environment variable ABC to xyz inside running container
* * --publish X:Y / -p X:Y <- forward local port X to port Y inside container
* * -i <-- enter interactive mode, commonly used with -t (create tty)
* * --rm <-- remove container after exiting command
* docker container stats <- statistics of running containers
* docker container inspect XXX <- details (ex. config) of running container XXX
* docker container logs XXX <- show console logs from running container XXX
* docker container network <- subcommand for virtual networks, it's good to create one fot each application (functionality)
* * --net=host <- skipping virtual network for container, using host network
* * docker container port XXX <- show ports forwarded to (published by) container XXX
* * dockert xyz inspect XXX <- informations (JSON) about XXX object (of xyz type)

## Elements of Dockefile

* RUN <- command(s) to be executed inside image
* * each RUN creates one layer
* containerized application log files should be linked to /dev/stdout and|or /dev/stderr to allow logs reading
* EXPOSE informs about ports used by application inside container, does not automatically publish port
* docker image build -t \<tagname\> . <-- build image in current directory with tag \<tagname\>
* the most often changed lines of dockerfile should be at the bottom no rebuild only layers related with these files and not touch previous ones

## Permanent storage

VOLUME in Dockerfile informs where application data are stored

Two ways of mounting:

* volume file (docker object)
* bind mount (link to host filesystem)

Examples:

```shell
docker container run ... -v /path/from/volume/in/dockerfile imagename # unnamed volume
docker container run ... -v somename:/path/from/volume/in/dockerfile imagename # volume named somename
docket container run ... -v /some/local/path:/path/from/volume/in/dockerfile imagename # binding to host filesystem
```
docker volume <- set of commands for volume management

## Docker compose

docker-compose.yml <- default yaml file with full environment description (images, volumes, networks)
docker-compose up <- setup environment, create/launch everything
docker-compose down <- stop everything and remove networks, volumes and so

## Swarm (orchestration, containers management, scalling)

docker info | grep Swarm <- shows current Swarm status

docker swarm init <- initialize single node swarm on host
(sample result: docker swarm join --token SWMTKN-1-5wsbbhjmmiqw1qygzqaxq6wi8q4t7g6cmf91gctm0ugyyfp7cq-86zvlqylxfqb45jagnoir80sz 192.168.163.113:2377)

docker service .... --detach true <- detach command, do not wait for it's execution (ex. when startig service or creating replicas), detach is false by default starting from 17.12

docker-machine create <somename>

docker swarm init --advertise-add X.Y.V.Z <- run docker swam with listening on specific interface, to be launched on one done

docker swarm join --token xxxxx X.Y.V.Z:<port> <- join cluster, to be launched on other nodes than first one

docker node ... <- node managing/listing commands, works only on manager node (locked on worker node)

docker network create -d overlay <name> <- create an overlay network (across all swarm nodes)

## Stack - launching services in Swarm based environment on yaml file (compose file)

```yaml
version: "3" <- needed at least compose v 3 to properly support stack (i.e. deploy section with replicas option and so)

services:
  servicename:
    image: <somename>:<sometag>
    ports:
      - "1234"
    networks:
    depends_on:
    ....
    ....
    deploy:
      replicas: 3
      update_config:
        parallelism: 2
        delay: 10s <- for services with some long warm up time
      restart_policy:
        condition: on-failure
        max_attepts: 3 <- try to restart only 3 times
        window: 120s
      placement:
        constraints: [node.role == manager]
```

docker stack deploy -c compose-file.yml <name> <- launch stack described in compose-file.yml, stack name will be <name>; can be used also to update stack after compose-file.yml update

docker stack <subcommand> <- stack management

docker stack services <name> <- list services in stack <name> with number of replicas and image names

## Swarm secrets (v > 1.13)

* secrets are mounted in /run/secrets/<secret_name> or /run/secrets/<secret_alias> and can be used only inside swarm
* there's a workaround to use file-based secrets in docker-compose (not the same as above, not secure)

* to use secrets in Stack there's yaml at least 3.1 needed:

version: "3.1"

services:
  psql:
    image: <someimage>:<somename"
    secrets:
      - secret_1
      - secret_2
    environment:
      SOME_VALUE_FILE: /run/secrets/secret_1
      OTHER_VALUE_FILE: /run/secrets/secret_2

secrets:
  secret_1:
    file: ./plaintextfile1
  secret_3:
    file: ./plaintextfile2

## Managing different configuration using docker-compose

docker-compose -f file1.yml -f file2.yml config > output.yml <- consolidates two files and creates one output to be used i.e. in production


## Other infos:

* stack deploy will perform an update operation if called again for already created stack
docker stack deploy -c file.yml <servicename> <- deploys new or updates already launched service

docker service scale service1=X service2=y <- change number of replicas of two servises to X and Y

* HEALTHCHECK allows to perfom basic check whether container started as expected (ex. server is launched on propor port, did not exited)

* container healthy status can be checked using
docker container ls
* and
docker container inspect

* USER option in Dockerfile can (and should) be used to avoid launching apps in container as root
* when using user then copy application files with chow:
COPY --chown=user:group /local/source /container/destination

## Docker Registry

* Tool for creating private "Docker Hub" with local storage
* By default works on port 5000
* It can be used to create some kind of proxy with local cache to save a copy of each image downloaded from Docker Hub - usefull when having multiple servers running docker

docker container run -d -p 5000:5000 registry <- use docker to launch... docker registry
docker container run -d -p 5000:5000 -v <volume_definition>:/var/lib/registry

docker tag <image_name> <registry_ip>:<port>/<newtag>
docker push <registry_ip>:<port>/<newtag>

## Other docker tools

* guthub.com/docker/docker-bench-security <- Docker configuration validator/checker


## Kubernetes

Is composed of:

* Control plane (master servers)
* Nodes (also called workers)

Each master is launched on top of Docker (or other containerization engine). It runs: etcd, API, scheduler, controller manager, core DNS). Each node is also launched on top of Docker and it runs: kubelet (K8s agent), kube-proxy (for networking)

### Differences of kubectl version <1.18 and >= 1.18

kubectl run nginx --image nginx <- creates a Deployment named nginx before 1.18 (which creates a ReplicaSet, which creates a Pod)
kubectl run nginx --image nginx <- creates a Pod named nginx in 1.18+
kubectl create deployment nginx --image nginx <- creates a Deployment in 1.18:

### Services in K8s

Service in K8s is a stable (unchanged) address for POD(s)

Service types in K8s:

* ClusterIP
* NodePort
* LoadBalancer
* ExternalName

kubectl expose deployment/httpenv --port 8888 <- expose port (like publish in Docker) but internally, inside cluster
kubectl expose deployment/httpenv --port 8888 --name <\somename\> --type NodePort <- expose outside cluster, ex. access from host

### K8s configuration yaml

Top level sections of K8s YAML file:

* apiVersion <- version for the kind - not for the file or K8s, ex. for 'kind: Deployment' it can be 'apiVersion: apps/v1' while for 'kind: Pod' just 'apiVersion: v1'
* kind <- type of object described, ex: Pod, Deployment, Service
* metadata
* spec

### K8s yaml creation

kubectl api-resources <- lists all available API resources with their 'kinds' and 'apigroups' (to be used in apiVersion section)
kubectl api-versions <- lists versions of available apigroups

kubectl explain services <- shows main section of yaml file for service
kubectl explain services --recursive <- shows yaml structure with all possible/mandatory (sub)fields for service
kubectl explain services.spec <- show fields for 'spec' section of yaml file for service kind

## Labels and Annotations in K8s yaml file

* Both types of additional description can be placed inside Metadata section
* Labels is for simple key-value pairs (ex. env: prod, customer: somecompany.com
* Annotations is for more complex data storage

* Labels can be used for filering (ex by kubectl)
kubectl get pods -l app=nginx <- show all pods that have label app set to nginx
kubectl apply -f somefile.yml -l app=nginx <- apply given yaml file but only objects with given label
