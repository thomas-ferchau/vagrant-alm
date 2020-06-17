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

  # vagrant up tower --provider virtualbox
  config.vm.define "tower" do |tower|
      tower.vm.hostname = "tower.local"
      tower.vm.box = "ansible/tower"
      tower.vm.network :private_network, ip: "192.168.56.2"
      tower.vm.provider :virtualbox do |v|
         v.gui = false
         v.memory = 2048
         # Nested Virtualization in Virtualbox funktioniert nur mit 1 CPU in den inneren VMs:
         # https://www.virtualbox.org/ticket/19561
         #v.cpus = 2
         v.cpus = 1
      end
      tower.vm.provider :libvirt do |lb|
          lb.memory = 2048
      end
  end

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://atlas.hashicorp.com/search.
  config.vm.box = "centos/7"

  # give boxes more time to boot - especially required for nested virtualization
  config.vm.boot_timeout = 1200

  # TODO: let's refactor and build a function for god's sake

  config.vm.define "jenkins" do |jenkins|
      jenkins.vm.hostname = "jenkins.local"
      jenkins.vm.network :private_network, ip: "172.16.10.100"
      jenkins.vm.provider :virtualbox do |v|
         v.gui = false
         v.memory = 2048
      end
      jenkins.vm.provider :libvirt do |lb|
          lb.memory = 2048
      end
  end
    
  config.vm.define "sonar" do |sonar|
    sonar.vm.hostname = "sonar.local"
    sonar.vm.network :private_network, ip: "172.16.10.110"
    sonar.vm.provider :virtualbox do |v|
        v.gui = false
        v.memory = 3000
        # Nested Virtualization in Virtualbox funktioniert nur mit 1 CPU in den inneren VMs:
        # https://www.virtualbox.org/ticket/19561
        #v.cpus = 2
        v.cpus = 1
    end
    sonar.vm.provider :libvirt do |lb|
      lb.memory = 2048
    end
 end

  config.vm.define "nexus", primary: true do |nexus|
    nexus.vm.hostname = "nexus.local"
    nexus.vm.network :private_network, ip: "172.16.10.120"
    nexus.vm.provider :virtualbox do |v|
        v.gui = false
        v.memory = 1024   
    end
    nexus.vm.provider :libvirt do |lb|
        lb.memory = 1024
    end
  end

  config.vm.define "app", primary: true do |app|
    app.vm.hostname = "app.local"
    app.vm.network :private_network, ip: "172.16.10.130"
    app.vm.provider :virtualbox do |v|
        v.gui = false
        v.memory = 512
    end
    app.vm.provider :libvirt do |lb|
        lb.memory = 512
    end
  end

  config.vm.define "app2", primary: true do |app2|
    app2.vm.hostname = "app2.local"
    app2.vm.network :private_network, ip: "172.16.10.140"
    app2.vm.provider :virtualbox do |v|
        v.gui = false
        v.memory = 512
    end
    app2.vm.provider :libvirt do |lb|
        lb.memory = 512
    end
  end

  config.vm.define "ansiblehost", primary: true do |ansiblehost|
    ansiblehost.vm.hostname = "ansiblehost.local"
    ansiblehost.vm.network :private_network, ip: "172.16.10.190"
    ansiblehost.vm.provider :virtualbox do |v|
        v.gui = false
        v.memory = 512
    end
    ansiblehost.vm.provider :libvirt do |lb|
        lb.memory = 512
    end

    # Rsync the current folder to /vagrant on ansiblehost (exclude ansible/roles to prevent downloaded roles from being deleted every time)
    ansiblehost.vm.synced_folder ".", "/vagrant", type: "rsync", rsync__verbose: true, rsync__exclude: ["ansible/roles"]

    # Copy .vagrant directory (excluded from rsync by default)
    ansiblehost.vm.provision "file", source: ".vagrant", destination: "/vagrant/.vagrant"
    ansiblehost.vm.provision "shell", inline: "chmod og-rwx /vagrant/.vagrant/machines/*/*/private_key"

    ansiblehost.vm.provision "ansible_local" do |ansible|
      # See alm.yml on how to run the ansible provisioning manually for a specific type of hosts
      ansible.playbook = "/vagrant/ansible/alm.yml"
      ansible.galaxy_role_file = "/vagrant/ansible/requirements.yml"
      ansible.limit = "all"
      ansible.inventory_path = "/vagrant/ansible/inventory.ini"
      ansible.provisioning_path = "/vagrant/ansible" # required for ansible.cfg
      ansible.groups = {
          "jenkins_server" => ["jenkins"],
          "sonar_server" => ["sonar"],
          "nexus_server" => ["nexus"],
          "app_server" => ["app", "app2"],
      }
      # For Debugging in case of errors:
      # ansible.verbose = "vvv"
    end
  end

  config.vm.provider "libvirt" do |libvirt|
      libvirt.storage_pool_name = "ext_storage"
  end

  if Vagrant.has_plugin?("vagrant-hostmanager")
      config.hostmanager.enabled = true
      config.hostmanager.manage_host = true
      config.hostmanager.manage_guest = true
  end
end
