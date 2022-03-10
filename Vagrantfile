#!/usr/bin/env ruby
Vagrant.configure("2") do |config|
    config.vm.box = "ubuntu/focal64"
    config.ssh.forward_agent = true
  
    inventory_name = ENV.fetch("INVENTORY", "test-inventory")
    tags = ENV.fetch("TAGS", "all")
    playbook = ENV.fetch("PLAYBOOK", "bernardko.coord.setup")
    extra_files = File.expand_path("./#{inventory_name}/files/", File.dirname(__FILE__))

    # Provide enough RAM to build python 3
    config.vm.provider :virtualbox do |virtualbox|
      virtualbox.customize ["modifyvm", :id, "--memory", 2048]
    end
  
    config.vm.define :vagranthost do |node|
      node.vm.box = "ubuntu/focal64"
      node.vm.hostname = "vagranthost"
      node.vm.network :forwarded_port, guest: 80, host: 8080
      node.vm.network :private_network, ip: "192.168.139.100"
      node.vm.provision :ansible do |ansible|
        ansible.playbook = File.expand_path("vagrant.yml")
        ansible.inventory_path = "./#{inventory_name}/vagrant"
        ansible.extra_vars = { env:"vagrant", playbook_name:playbook, inventory_name:inventory_name, files:extra_files }
        ansible.galaxy_role_file = "requirements.yml"
        ansible.config_file = "ansible.cfg"
        ansible.tags=tags
        
        # Cannot have spaces in your raw_arguments - fails without errors
        # if using expand_path, check for spaces in the generated path
        ansible.raw_arguments = ["--extra-vars=@./#{inventory_name}/vagrant.vault"]
      end
    end
  end