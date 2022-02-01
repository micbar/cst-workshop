# oCIS with Docker

## Docker

### why Docker

or: why containers

- containers are more lightweight than VMs (but share resources)
- images have all dependencies installed
- easier dependency  management
- scaling, run multiple instances of a container, distribute them across hosts (eg. docker swarm)


### why docker-compose

- for deployments you often have multiple services (containers)
- the containers have volumes, networks, ...

- you will start writing bash scripts to create and update your containers

- BUT: that's what docker-compose does for you

- docker-compose describes the desired state of the docker container setup and takes actions to reach that state

- put your docker-compose files into a version control system!


### outline

- kubernetes are also just containers
- huge ecosystem:
  - service discovery
  - networking between hosts

## oCIS

### why oCIS with containers

- scale services differently vs. scale monoliths
