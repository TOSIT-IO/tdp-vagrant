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

# Helper functions
def destructure_host_dict(dict)
  dict.values_at("ip", "hostname", "cpus", "memory", "box")
end

domain = settings["domain"]
all_hosts = settings["tdp_manager_enabled"] ? (settings["hosts"] + settings["tdp_manager_host"]) : settings["hosts"]

# define groups and hostvars for ansible provisionner
ansible_configuration = all_hosts.each_with_object({"hostvars": {}, "groups": Hash.new {|hash, key| hash[key] = []}}) do |host, configuration|
  ip, hostname, cpus, memory, box = destructure_host_dict(host)
  configuration[:hostvars][hostname] = {
    :ip     => ip,
    :domain => domain,
  }
  host.fetch("groups", []).each do |group|
    configuration[:groups][group] << host["hostname"]
  end # end group
end # end configuration


Vagrant.configure("2") do |config|
  config.vm.synced_folder "./", "/vagrant", disabled: true

  all_hosts.each do |host|
    ip, hostname, cpus, memory, box = destructure_host_dict(host)
    box ||= settings["box"]
    config.vm.define hostname, autostart: true do |cfg|
      cfg.vm.box = box
      cfg.vm.hostname = hostname
      cfg.vm.network "private_network", ip: ip

      cfg.vm.provider :virtualbox do |vb|
        vb.name = hostname # sets gui name for VM
        vb.customize ["modifyvm", :id, "--memory", memory, "--cpus", cpus, "--hwvirtex", "on"]
        # Forward TDP UI and server port
        if hostname == "tdp-manager" && settings["tdp_manager_enabled"]
          cfg.vm.network "forwarded_port", guest: 3000, host: 3000, id: 'tdp-ui'
          cfg.vm.network "forwarded_port", guest: 8000, host: 8000, id: 'tdp-server'
        end # end if 

      end # end provider virtualbox

      cfg.vm.provider :libvirt do |libvirt|
        libvirt.cpus = cpus
        libvirt.memory = memory
        libvirt.qemu_use_session = false
        # Forward TDP UI and server port
        if hostname == "tdp-manager" && settings["tdp_manager_enabled"]
          cfg.vm.network "forwarded_port", guest: 3000, host: 3000, id: 'tdp-ui'
          cfg.vm.network "forwarded_port", guest: 8000, host: 8000, id: 'tdp-server'
        end # end if 
      end # end provider libvirt
    end # end define
  end # end settings

  config.vm.provision :ansible do |ansible|
    ansible.playbook = "#{vagrantfile_dir}/provision/main.yml"
    ansible.host_vars = ansible_configuration[:hostvars]
    ansible.groups = ansible_configuration[:groups]
    ansible.extra_vars = {tdp_manager_enabled: settings["tdp_manager_enabled"]}
    ansible.config_file = "#{vagrantfile_dir}/ansible.cfg"
  end # end provision
end
