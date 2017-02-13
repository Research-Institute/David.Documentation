# Creating the Infrastructure in Azure

## Pre-Requisites

- Docker
- Docker-Compose
- Docker-Machine

## Provisioning new machines

- Provision the Docker Swarm Manager using the [Azure driver](https://docs.docker.com/machine/drivers/azure/)

```
docker-machine create \
--driver azure \
--azure-location centralus \
--azure-subscription-id ${SUBSCRIPTION_ID} \
--azure-resource-group ${RESOURCE_GROUP} \
${MANAGER_NAME}
```

- SSH into the Swarm Manager

```
docker-machine ssh ${MANAGER_NAME}
```

- Initialize the Swarm. You need to use the private IP address (note this is NOT the IP shown in docker-machine ls). 
 - Get the private IP of the manager through the Azure management portal: 
   - (Resource Groups) -> (group you created) -> (docker-machine-vnet)
   - Look for the network interface (manger-nic) and grab the IP Address off that interface.
 - Run the following command with the private IP and copy the results.

```
docker swarm init
```

- Provision the worker node(s)

Run the same command used to create the manager, but change the machine name

- Join the swarm
  - SSH into the worker(s)
  - Join the swarm by pasting the command from the swarm initialization step

## Persistent Storage

There are a few options for persisting storage:

- Remotely hosted databases (we support SQL Server and PostgreSQL)
- PostgreSQL on Azure Disks
- Storage Accounts using the Azure File Docker Volume Driver (not yet recommended)

### Remotely Hosted Databases

Configure these using the connection string environment variables

### Azure VHD Disks

[Reference](https://docs.microsoft.com/en-us/azure/virtual-machines/virtual-machines-linux-classic-attach-disk)

- Create a disk using the Azure Management Portal or by using the Azure CLI
- Initialize the disk
  - SSH into the VM
  - Get the VHD identifier `sudo grep SCSI /var/log/syslog`
  - Create the device `sudo fdisk /dev/sdc`
  - Create partition: `n`
  - Make partition primary: `p`
  - Make it the first partition: `1`
  - See details: `p`
  - Write the settings: `w`
  - Create the file system: ` sudo mkfs -t ext4 /dev/sdc1`
  - Make directory to mount the system: `mkdir /david`
  - Mount the drive: `sudo mount /dev/sdc1 /david`
  - Add the new drive to /etc/fstab so that it survives restarts
    - Get the UUID `sudo -i blkid`
    - Open the fstab file: `sudo nano /etc/fstab`
    - Append to end of file: `UUID=33333333-3b3b-3c3c-3d3d-3e3e3e3e3e3e   /datadrive   ext4   defaults,nofail   1   2`
    - Test the mounting: `sudo umount /david` `sudo mount /david`
    - Make drive writeable: `sudo chmod go+w /david`
- Install [Docker local-persist volume plugin](https://github.com/CWSpear/local-persist)

### Configuring Storage Accounts [Not Yet Recommended]

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
 
 ## Adding a machine to CI
 
- (gitlab) Setup shell runner with Docker capabilities (https://docs.gitlab.com/ce/ci/docker/using_docker_build.html)
- (gitlab) sudo -u gitlab-runner bash
- Create a new key on your build machine

```
ssh-keygen -t rsa
```

- Add the key to the authorized hosts file on manager

- Verify you can SSH into the manager from the new machine


- Install docker-machine on the build machine without sudo:

```
curl -L https://github.com/docker/machine/releases/download/v0.9.0/docker-machine-`uname -s`-`uname -m` > ~/docker-machine && chmod +x docker-machine
```

- (gitlab)add to PATH 
```
PATH=$PATH:/home/gitlab-runner
```


- (gitlab)explicitly set the storage PATH
```
export MACHINE_STORAGE_PATH=/home/gitlab-runner/.docker
```

- create the machine

```
docker-machine create \
--driver generic  \
--generic-ip-address 13.88.25.6  \
--generic-ssh-key ~/.ssh/id_rsa \
--generic-ssh-user docker-user  \
manager
```
