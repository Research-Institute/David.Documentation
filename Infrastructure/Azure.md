# Creating the Infrastructure in Azure

## Pre-Requisites

- Docker
- Docker-Compose
- Docker-Machine

## Provisioning the machines

1. Provision the Docker Swarm Manager using the [Azure driver](https://docs.docker.com/machine/drivers/azure/)

```
docker-machine create --driver azure --azure-subscription-id ${SUBSCRIPTION_ID} --azure-resource-group ${RESOURCE_GROUP} ${MANAGER_NAME}
```