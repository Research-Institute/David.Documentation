# Creating the Infrastructure in Azure

## Pre-Requisites

- Docker
- Docker-Compose
- Docker-Machine

## Provisioning new machines

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
   - (Resource Groups) -> (group you created) -> (docker-machine-vnet)
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
  
## Allocating Existing Machines 

TODO

## Configuring Storage Accounts [Not Yet Recommended]

- Create an Azure Storage Account
- SSH into each machine in the Swarm
- Install `azurefile-dockervolumedriver` [ref](https://github.com/Azure/azurefile-dockervolumedriver/blob/master/contrib/init/systemd/README.md)
```
sudo -s
wget https://raw.githubusercontent.com/Azure/azurefile-dockervolumedriver/master/contrib/init/systemd/azurefile-dockervolumedriver.default
wget https://raw.githubusercontent.com/Azure/azurefile-dockervolumedriver/master/contrib/init/systemd/azurefile-dockervolumedriver.service
wget -qO/usr/bin/azurefile-dockervolumedriver https://github.com/Azure/azurefile-dockervolumedriver/releases/download/v0.5.1/azurefile-dockervolumedriver
chmod +x /usr/bin/azurefile-dockervolumedriver
mv azurefile-dockervolumedriver.default /etc/default/azurefile-dockervolumedriver
# add Azure configuration
nano /etc/default/azurefile-dockervolumedriver
mv azurefile-dockervolumedriver.service /etc/systemd/system/azurefile-dockervolumedriver.service
systemctl daemon-reload
systemctl enable azurefile-dockervolumedriver
systemctl start azurefile-dockervolumedriver
systemctl status azurefile-dockervolumedriver
```
- Create volumes on each machine (until this is [fixed](https://github.com/Azure/azurefile-dockervolumedriver/issues/81))
```
docker volume create -d azurefile -o share=doctordb
```

**Issues**

- Storage accounts must be in the same region as the VM
- Verify deployment specifies the `azurefile` driver for the volumes
- `initdb: could not change permissions of directory "/var/lib/postgresql/data": Operation not permitted` : see these issues [azurefile-dockervolumedriver#32](https://github.com/Azure/azurefile-dockervolumedriver/issues/32), [docker-library/postgres#235](https://github.com/docker-library/postgres/issues/235), [azurefile-dockervolumedriver#65](https://github.com/Azure/azurefile-dockervolumedriver/issues/65)

Also, from [azurefile-dockervolumedriver#32](https://github.com/Azure/azurefile-dockervolumedriver/issues/32#issuecomment-227664566):

> We currently have no volume drivers on Azure that would let you smoothly and safely run a database unfortunately. I have provided a bit of detail in my comments above. Essentially, creating a durable and robust container persistence solution would require block devices support and the current offerings are not meeting the demands. However there is work going on to fix this gap.
 
