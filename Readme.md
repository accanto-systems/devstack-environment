# Vanilla Devstack

This project creates an instance of an "All on One" vanilla Devstack in a virtual machine. The Devstack instance has neutron enabled and binds Openstack floating ip's to the Vagrant machine's external interface, making any virtual machines created inside the DevStack virtual machine accessible outside to the host machine. Ubuntu Bionic and Xenial images will be automatically added to the DevStack instance during the installation process. 

The Vagrantfile provided supports running Devstack on virtualbox or libvirt (QEMU/KVM). Running on a linux host that can provide KVM passthrough to the virtual machine has significant performance benefits.

# Configuring Devstack for your environment

## Vagrant Networking

If you are not using Vagrant to create your virtual machine you can skip this section. 

The provided script creates a single network interface. You can choose to create a private network that is internal (NAT'd to outside world) to the physical host or a public network that is bridged to one of the physical hosts interfaces (making openstack and all VMs inside available on you LAN).

Sample private network configuration in Vagrantfile
```
Vagrant.configure(2) do |config|
  config.vm.define 'devstack' do |nodeconfig|
    ...
    nodeconfig.vm.network 'private_network', ip: '192.168.56.130'
```

Sample public network configuration in Vagrantfile, you may be prompted for which network interface to bind to. The ip address specified must be appropriate for the CIDR on the LAN the physical machine is connected to.
```
Vagrant.configure(2) do |config|
  config.vm.define 'devstack' do |nodeconfig|
    ...
    nodeconfig.vm.network 'public_network', ip: '192.168.1.130'
```

## Ansible Configuration

Once you know your virtual machines network subnet/CIDR, you will need to reflect this in the variables.yml by updating the following ansible variables:
* public_interface: should be eth1 for virtual machines created by Vagrant because Vagrant uses eth0 for mgmt/ssh traffic, else set as appropriate for your machine
* gateway: the gateway ip address of the private/public network you have chosen
* floating_range: the CIDR for the floating ip addresses allocated to the network you have chosen
* start_floating: start of the address range
* end_floating: end of the address range
* volume_size: Amount of volume storage to allocate to the virtual machine in MBs

Update the inventory.yml file with your target virtuaml machine address and login credentials.

# Starting Devstack

## Vagrant

To create the DevStack VM with Vagrant, run the following command in a terminal
```
vagrant up
```

## Ansible

If you have a pre-existing virtual machine you can run the ansible playbooks to install devstack as follows:

```
ansible-playbook --inventory-file=inventory.yml setup-devstack.yml
```

This will take at least 20 mins depending your your internet connection. 

If you want to see progress, ssh to your virtual machine and run the following command:
```
tail -f /opt/stack/logs/stack.sh.log
```
This will show you current progress and when installation is complete

# Accessing the OpenStack dashboard

You should now be able to connect to OpenStack at http://<YOUR_IP>/dashboard. Username: "admin", password: "secret"

Note: If you cannot connect to the machine, ssh on and check that the virtual machine public NIC (e.g. eth1 on Vagrant) does NOT have an IP address. If it has an ip address, run the following command to zero it. 

```
ifconfig eth1 0
```

