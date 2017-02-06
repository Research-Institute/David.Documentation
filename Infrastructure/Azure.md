# Creating the Infrastructure in Azure

## Pre-Requisites

- Docker
- Docker-Compose
- Docker-Machine

## Provisioning the machines

- Provision the Docker Swarm Manager using the [Azure driver](https://docs.docker.com/machine/drivers/azure/)

```
docker-machine create --driver azure --azure-subscription-id ${SUBSCRIPTION_ID} --azure-resource-group ${RESOURCE_GROUP} ${MANAGER_NAME}
```

- SSH into the Swarm Manager

```
docker-machine ssh staging-manager
```

- Initialize the Swarm. You need to use the private IP address (note this is NOT the IP shown in docker-machine ls). 
 - Get the private IP of the manager through the Azure management portal: 
   - (Resource Groups -> (group you created) -> docker-machine-vnet
   - Look for the network interface (manger-nic) and grab the IP Address off that interface.
 - Run the following command with the private IP and copy the results.

```
docker swarm init --advertise-addr ${MANAGER_PRIVATE_IP}:2376
```


- Provision the worker node(s)

```
docker-machine create --driver azure --azure-subscription-id ${SUBSCRIPTION_ID} --azure-resource-group ${RESOURCE_GROUP} ${WORKER_NAME}
```

- Join the swarm
  - SSH into the worker(s)
  - Join the swarm by pasting the command from the swarm initialization step