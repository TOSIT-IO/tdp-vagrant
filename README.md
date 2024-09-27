# TDP Vagrant

Launch a fully-featured virtual TDP Hadoop cluster with a single command _or_ customize the infrastructure.

## Requirements

- Vagrant >= 2.2.19 (to launch and manage the VMs)
- Ansible to provision the VMs.

You must choose a provider to launch VMs. By default, Vagrant use Virtualbox.

### Virtualbox provider

- VirtualBox >= 6.1.26

This is the default provider, no plugin needed.

### Libvirt provider

- Vagrant libvirt plugin (vagrant-libvirt) >= 0.9.0

Follow documentation to install it <https://vagrant-libvirt.github.io/vagrant-libvirt/>.

## Launch cluster

```bash
# With default virtualbox provider
vagrant up
# With libvirt provider
vagrant up --provider=libvirt
```

You can change the default provider with environment variable.

```bash
export VAGRANT_DEFAULT_PROVIDER=libvirt
vagrant up
```

**Important:** The Vagrantfile create an internal network so you must not run Vagrant in parallel because the internal network can be created multiple times leading to undefined behavior. With the Libvirt provider, VMs are launch in parallel so, if you want speed, use Libvirt provider.

## Connect to machine

```bash
# Connect to edge
vagrant ssh edge-01
```

## Destroy cluster

```bash
vagrant destroy
```

For virtualbox provider the destroy command does not destroy the internal network.
This network use an interface on the host `vboxnet<N>` which need to be delete manually.

```bash
# https://www.virtualbox.org/manual/ch06.html#network_hostonly
VBoxManage hostonlyif remove vboxnet0
```

## Containerized launch

The Docker container has a Libvirt client installed which launches the VMs on your host. The only requirement is to have Docker and Libvirt.

- Check the Libvirt GID for your user the command `id`.

- Build the image:

```sh
docker build -t tdp-vagrant Docker
```

- Run and enter the container with a bind mount to the current host directory :

```sh
docker run -it --name tdp-vagrant --rm \
-w /home/tdp/tdp-dev \
-v $PWD:/home/tdp/tdp-dev \
-v /var/run/libvirt/libvirt-sock:/var/run/libvirt/libvirt-sock \
-v vagrant-boxes:/home/tdp/.vagrant.d/boxes \
--env CONTAINER_UID=$(id -u) --env CONTAINER_GID=$(getent group libvirt | cut -d: -f3) \
tdp-vagrant bash
```

You may run the vagrant commands to launch, ssh and destroy the VMs previously described.

Vagrant with the Ansible provisioning creates the inventory file `.vagrant/provisioners/ansible/inventory/vagrant_ansible_inventory`. The path of the SSH keys is always absolute, consequently the starting point is the root path of the container and not of your host.

Since the container has ansible-core==2.15.1 installed and required for TDP collections, you may launch TDP playbooks from the container. Make sure you can access them through a bind mount.

As The VMs are on your host, you may use Libvirt commands to manage them there.
