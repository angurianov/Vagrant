# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.
  config.vm.box = "generic/ubuntu2004"
  config.vm.network "forwarded_port", guest: 3000, host: 3000
#  config.vm.synced_folder ".", "/vagrant", create: true
  config.vm.synced_folder ".", "/vagrant"
#  config.vm.provision "docker" do |d|
#
#  end

# Run Ansible from the Vagrant VM
  config.vm.provision "ansible_local" do |ansible|
    ansible.playbook = "playbook.yml"
    ansible.groups = {
        "localhost" => ["default"],
        "dev_enviroment" => ["default"]
    }

  end


end
