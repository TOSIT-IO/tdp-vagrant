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

## Setup python environment

```bash
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

## Launch cluster

If the `tdp_manager_enabled` flag in the vagrant.yml file is set to true, vagrant will provision a virtual machine for the TDP Manager, which is responsible for deploying the [tdp-getting-started](https://github.com/TOSIT-IO/tdp-getting-started) cluster.

```bash
# Activate Python virtual env
source ./venv/bin/activate
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

## TDP Quick Start

If the `tdp_manager_enabled` flag is set to true, follow these steps:

### TDP Prerequisites
```bash
# Connect to tdp-manager
vagrant ssh tdp-manager
# Move into project dir
cd /opt/tdp/tdp-getting-started
# Activate Python virtual env and set environment variables
source ./venv/bin/activate && source .env
# Configure TDP prerequisites
ansible-playbook ansible_collections/tosit/tdp_prerequisites/playbooks/all.yml
```

## Deploy TDP 
You have multiple options to deploy a TDP cluster:
### Option 1: Deploy with TDP UI

Access the TDP UI at http://localhost:3000/ on your local machine.

- In the UI, go to "Deployments".
- Select "New deployment", "Deploy from the DAG", and "Preview" (by default all services are deployed).
- Click "Deploy."

You can see the deployment in the "Deployments" page. Wait deployment to complete.

### Option 2: Deploy with TDP server API

You can access the TDP server at http://localhost:8000/ on your local machine.

Run the following command to deploy TDP:

```bash
# Deploy TDP cluster core and extras services
curl -X POST http://localhost:8000/api/v1/deploy/dag
```

To check the deployment status:

```bash
# run in your local machine
while ! curl -s http://localhost:8000/api/v1/deploy/status | grep -q "no deployment on-going"; do sleep 10; done
# Connect to tdp-manager
vagrant ssh tdp-manager
# Check deployment logs
sudo journalctl -u tdp-server.service -f
# Wait for deployment to complete
```

### Option 3: Deploy with TDP lib CLI

```bash
# Connect to tdp-manager
vagrant ssh tdp-manager
# Move into project dir
cd /opt/tdp/tdp-getting-started
# Activate Python virtual env and set environment variables
source ./venv/bin/activate && source .env
# Deploy TDP cluster core and extras services
tdp deploy
```

### Option 4: Deploy with Ansible Playbook

```bash
# Connect to tdp-manager
vagrant ssh tdp-manager
# Move into project dir
cd /opt/tdp/tdp-getting-started
# Activate Python virtual env and set environment variables
source ./venv/bin/activate && source .env
# Deploy TDP cluster core services
ansible-playbook ansible_collections/tosit/tdp/playbooks/meta/all.yml
# Deploy extras services
ansible-playbook ansible_collections/tosit/tdp_extra/playbooks/meta/livy.yml
ansible-playbook ansible_collections/tosit/tdp_extra/playbooks/meta/livy-spark3.yml
ansible-playbook ansible_collections/tosit/tdp_extra/playbooks/meta/zookeeper-kafka.yml
ansible-playbook ansible_collections/tosit/tdp_extra/playbooks/meta/kafka.yml
# Deploy observability services
ansible-playbook ansible_collections/tosit/tdp_observability/playbooks/meta/prometheus.yml
ansible-playbook ansible_collections/tosit/tdp_observability/playbooks/meta/grafana.yml
```
## Post-Installation

After deploying the TDP cluster, perform the following post-installation tasks:

```bash
# Connect to tdp-manager
vagrant ssh tdp-manager
# Move into project dir
cd /opt/tdp/tdp-getting-started
# Activate Python virtual env and set environment variables
source ./venv/bin/activate && source .env
# Configure HDFS user home directories
ansible-playbook ansible_collections/tosit/tdp/playbooks/utils/hdfs_user_homes.yml
# Configure Ranger policies
ansible-playbook ansible_collections/tosit/tdp/playbooks/utils/ranger_policies.yml
```

## Connect to machine

```bash
# Connect to edge
vagrant ssh edge-01
```

## Web UIs Links

- [HDFS NN Master 01](https://master-01.tdp:9871/dfshealth.html)
- [HDFS NN Master 02](https://master-02.tdp:9871/dfshealth.html)
- [YARN RM Master 01](https://master-01.tdp:8090/cluster/apps)
- [YARN RM Master 02](https://master-02.tdp:8090/cluster/apps)
- [MapReduce Job History Server](https://master-03.tdp:19890/jobhistory)
- [HBase Master 01](https://master-01.tdp:16010/master-status)
- [HBase Master 02](https://master-02.tdp:16010/master-status)
- [Spark History Server](https://master-03.tdp:18081/)
- [Spark3 History Server](https://master-03.tdp:18083/)
- [Ranger Admin](https://master-03.tdp:6182/index.html)
- [JupyterHub](https://master-03.tdp:8000/)

**Note:** All the WebUIs are Kerberized, you need to have a working Kerberos client on your host, configure the KDC in your `/etc/krb5.conf` file and obtain a valid ticket. 
You can also access the WebUIs through Knox:

- [HDFS NN](https://edge-01.tdp:8443/gateway/tdpldap/hdfs)
- [YARN RM](https://edge-01.tdp:8443/gateway/tdpldap/yarn)
- [MapReduce Job History Server](https://edge-01.tdp:8443/gateway/tdpldap/jobhistory)
- [HBase Master](https://edge-01.tdp:8443/gateway/tdpldap/hbase/webui/master/master-status?host=master-01.tdp&port=16010)
- [Spark History Server](https://edge-01.tdp:8443/gateway/tdpldap/sparkhistory)
- [Spark3 History Server](https://edge-01.tdp:8443/gateway/tdpldap/spark3history)
- [Ranger Admin](https://edge-01.tdp:8443/gateway/tdpldap/ranger)

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
