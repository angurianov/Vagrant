Vagrant.configure("2") do |config|

  config.vm.box = "generic/ubuntu2004"
  config.vm.network "forwarded_port", guest: 3000, host: 3000			# проброс порта для графаны
  config.vm.synced_folder ".", "/vagrant"					# создание общей папки
  config.vm.provision "file", source: ".", destination: "/home/vagrant"		# копирование файлов внутрь виртуалки

# Run Ansible from the Vagrant VM
  config.vm.provision "ansible_local" do |ansible|
    ansible.playbook = "playbook.yml"
    ansible.groups = {
        "localhost" => ["default"],						# имя виртуалки не указано, поэтому она будет фигурировать как default. Плейбук отработает без создания инвентори
        "dev_enviroment" => ["default"]
    }

  end

end
