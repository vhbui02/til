# Docker Swarm

## Cheatsheet

```sh

# ======================================================================= #
# MANAGER / WORKER                                                        #
# ======================================================================= #

# on manager node/host, initialize a swarm cluster
# technically, generate token so workers can join
docker swarm init

# display the token to join a manager/a worker
docker swarm join-token worker
docker swarm join-token manager

# on worker node/host, join a cluster managed by a manager node/host
docker swarm join --token <token> <manager-ip>:2377

# NOTE: manager node and worker node can be placed on the same network infrastructure.

# create network
docker network create -d overlay swarm-net

# ======================================================================= #
# SERVICE                                                                 #
# ======================================================================= #

# create a service
# microservice in a nutshell
# e.g. create 3 Nginx replicas for load balancing
docker service create --name web-server --replicas 3 --network swarm-net -p 8080:80 nginx

# test the service
curl <private-ip>:8080

# scale a service
docker service scale web-server=5

# no-downtime rolling update
docker service update --image nginx:1.29 web-server

# ======================================================================= #
# MONITORING                                                              #
# ======================================================================= #

# check swarm node/host
docker node ls

# debug
# check your containers running on which node/host
docker service ps web-server
docker service logs web-server
```

## Overview

First step in container orchestration (no, not Kubernetes [https://doineedkubernetes.com](https://doineedkubernetes.com))

**NOTE:**

### What is a swarm?

A swarm consists of multiple Docker hosts (hosts === physical machines/VMs/cloud servers) which run in Swarm Mode.

**NOTE:** Swarm Mode is different from "Classic Swarm", a depreciated version of Docker Swarm. Swarm Mode leverage `Docker Compose v3`.

A "Node" is an instance of the Docker Engine OS process. You can run multiple Docker Engine processes (a.k.a Nodes) on the same host (again, hosts === physical machines/VMs/cloud servers), but each is a separate OS process, **NOT A CONTAINER**. These Nodes can be managers, or workers.

- Managers: manage membership and delegation
- Workers: run swarm services.

Node runs containers. Node is not a container.

You can have one or more Nodes deployed on a host, but it's more recommended to deploy them distributedly across multiple hosts.

### Manager Node

When deploying your application to a swarm, you submit a "service definition" to a Manager Node. The Manager Node dispatches units of work called "Tasks" to Worker Nodes.

Manager Node performs the orchestration and cluster management functions required to maintain the desired state of the swarm.

Among Manager Nodes, a node is selected to be a single leader for orchestration task.

Manager Node can also run services as worker nodes

### Worker Node

Worker Node receives and executes Task from Manager Node. 

### Swarm Service

Service is the definition of the tasks to execute on the manager/worker nodes

Introduce the concept of

There are 2 types of nodes: Manager and Worker.

Workers run services.

### Manager

- One-or-many nodes that manage a cluster of worker nodes deploy on single host/multiple hosts.
- Using Raft consensus algorithm, multiple manager nodes can collectively manage the swarm state
- Although, one manager node is elected as the leader to handle certain operations.

### Worker

- Worker nodes execute tasks assigned by manager nodes.
- Workers run service containers and do not participate in swarm management decisions.

### Service (or Swarm Service, not to be confused with Docker Compose service)

- The definition of a containerized application that shows the desired state of the service

## Differences between Docker Swarm and Compose

- Docker Compose: multi-container on ONE host
- Docker Swarm: multi-container on MULTIPLE hosts.

## Pros in using Docker Swarm

- Dead simple.
- Fast scaling.
- Easy to build CI/CD pipeline.

## Real case study

- A small team use Swarm instead of K8s for a 10-services system, reduce time to on-board new tech and manage cluster by 50% and still can manage 1000 users at the same time.

## References

- [Docker Official Documentation, Swarm mode key concepts](https://docs.docker.com/engine/swarm/key-concepts/)
- [DevOps VN, "Bài 6. Docker Swarm: Orchestration Cơ Bản để Scale Containe"](https://devops.vn/posts/bai-6-docker-swarm-orchestration-co-ban-de-scale-container/)
