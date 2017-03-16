# Introduction

The services are running in a [Docker Swarm](https://docs.docker.com/engine/swarm/) cluster. 
In order to manage the services, you will need ot SSH into the manager node. 

Once you are logged into the manager, you can use [Docker Swarm](https://docs.docker.com/engine/swarm/#whats-next) 
service API to view services:

```
docker service ls # list the services
docker service scale {SERVICE_NAME}={INSTANCE_COUNT}
docker service ps {SERVICE_NAME} # view service instances and where they are deployed
```
