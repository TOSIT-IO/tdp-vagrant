# -*- mode: ruby -*-
# vi: set ft=ruby :

require 'yaml'

vagrantfile_dir = File.dirname(File.realpath(__FILE__))

if File.file?("./vagrant.yml")
  config_file = YAML.load_file("./vagrant.yml")
else 
  config_file = YAML.load_file("#{vagrantfile_dir}/vagrant.yml")
end
settings = config_file

Vagrant.configure("2") do |config|
  config.vm.box = settings["box"]
  config.vm.synced_folder "./", "/vagrant", type: "rsync", rsync__auto: true, rsync__exclude: ['files/*.tar.gz', 'files/*.tgz', 'collections/', 'group_vars/', 'logs/', 'roles/']

  settings["hosts"].each do |host_data|
    ip, hostname, domain, cpus, memory = *host_data
    config.vm.define hostname, autostart: true do |cfg|
      cfg.vm.hostname = hostname
      cfg.vm.network "private_network", ip: ip

      cfg.vm.provider :virtualbox do |vb|
        vb.name = hostname # sets gui name for VM
        vb.customize ["modifyvm", :id, "--memory", memory, "--cpus", cpus, "--hwvirtex", "on"]
      end # end provider virtualbox

      cfg.vm.provider :libvirt do |libvirt|
        libvirt.cpus = cpus
        libvirt.memory = memory
      end # end provider libvirt

    end # end define
  end # end settings

  config.vm.provision :ansible do |ansible|
    ansible.playbook = "#{vagrantfile_dir}/ping.yml"
    ansible.host_vars = settings["hosts"].each_with_object ({}) do |(ip, hostname, domain, cpus, memory), dict|
      dict[hostname] = {
        :ip     => ip,
        :domain => domain,
      }
    end # end host_vars
  end # end provision
end
