# -*- mode: ruby -*-
# vi: ft=ruby :

require 'rbconfig'
require 'yaml'
DEFAULT_BASE_BOX = "centos/7"
VAGRANTFILE_API_VERSION = '2'

cpuCap = 50                                   # Limit to 10% of the cpu
inventory = YAML.load_file("elastic-stack/inventory.yml")   # Get the names & ip addresses for the guest hosts


Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
    config.vm.synced_folder '.', '/elastic',
        owner: "vagrant",
        mount_options: ["dmode=775,fmode=600"],
        resync: true
    
    inventory.each do |group_k, grouphosts_v|
        grouphosts_v['hosts'].each do | hostname_k, hostinfo_v |
            config.vm.define hostname_k do | node |
                node.vm.box = hostinfo_v['box'] ||= DEFAULT_BASE_BOX
                node.vm.hostname = hostname_k
                node.vm.network :private_network, ip: hostinfo_v['ansible_host']
                ram = hostinfo_v['memory']
                node.vm.provider :virtualbox do |vb|
                    vb.name = hostname_k
                    vb.customize ["modifyvm", :id, "--cpuexecutioncap", cpuCap, "--memory", ram.to_s]
                    vb.cpus = 2
                end
            end
        end
    end

    config.vm.define 'controller' do |machine|
        machine.vm.box = DEFAULT_BASE_BOX
        machine.vm.hostname = "controller"
        machine.vm.network "private_network", ip: "192.168.33.20"
        machine.vm.provision :ansible_local do |ansible|
          ansible.playbook       = "/elastic/elastic-stack/0_install.yml"
          ansible.verbose        = true
          ansible.install        = true
          ansible.config_file    = "/vagrant/elastic-stack/ansible.cfg"
          ansible.limit          = "all" # or only "nodes" group, etc.
          ansible.inventory_path = "/elastic/elastic-stack/inventory.yml"
          ansible.galaxy_role_file = "/elastic/elastic-stack/requirements.yml"
          ansible.galaxy_command = "ansible-galaxy install --role-file=%{role_file} --force"
        end
    end
end

