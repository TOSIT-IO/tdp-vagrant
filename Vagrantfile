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

# define groups and hostvars for ansible provisionner
ansible_configuration = settings["hosts"].each_with_object({"hostvars": {}, "groups": Hash.new {|hash, key| hash[key] = []}}) do |host, configuration|
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

  settings["hosts"].each do |host|
    ip, hostname, cpus, memory, box = destructure_host_dict(host)
    box ||= settings["box"]
    config.vm.define hostname, autostart: true do |cfg|
      cfg.vm.box = box
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
    ansible.playbook = "#{vagrantfile_dir}/provision.yml"
    ansible.host_vars = ansible_configuration[:hostvars]
    ansible.groups = ansible_configuration[:groups]
  end # end provision
end
