# TDP Vagrant

Launch a fully-featured virtual TDP Hadoop cluster with a single command _or_ customize the infrastructure.

## Requirements

- Vagrant >= 2.2.19 (to launch and manage the VMs)

You must choose a provider to launch VMs. By default, Vagrant use Virtualbox.

### Virtualbox provider

- VirtualBox >= 6.1.26

This is the default provider, no plugin needed.

### Libvirt provider

- Vagrant libvirt plugin (vagrant-libvirt) >= 0.9.0

Follow documentation to install it https://vagrant-libvirt.github.io/vagrant-libvirt/.

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
