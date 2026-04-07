Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/bionic64"

  nodes = [
    { name: "k8s-master", ip: "192.168.56.30" },
    { name: "k8s-worker-1", ip: "192.168.56.31" },
    { name: "k8s-worker-2", ip: "192.168.56.32" },
    { name: "vm-tools", ip: "192.168.56.33" },
  ]

  nodes.each do |node|
    config.vm.define node[:name] do |node_config|
      node_config.vm.hostname = node[:name]
      node_config.vm.network :private_network, ip:node[:ip]

      if node[:name] == "vm-tools"
        node_config.vm.network "forwarded_port", guest: 8081, host: 8081, auto_correct: true
      end

      node_config.vm.provider "virtualbox" do |vb|
        vb.memory = "2048"
        vb.cpus = 2
        vb.name = node[:name]
      end

      node_config.vm.provision "shell", inline: <<-SHELL
        mkdir -p /home/vagrant/keys
        cp /vagrant/.vagrant/machines/k8s-master/virtualbox/private_key /home/vagrant/keys/k8s-master_key 2>/dev/null || true
        cp /vagrant/.vagrant/machines/k8s-worker-1/virtualbox/private_key /home/vagrant/keys/k8s-worker-1_key 2>/dev/null || true
        cp /vagrant/.vagrant/machines/k8s-worker-2/virtualbox/private_key /home/vagrant/keys/k8s-worker-2_key 2>/dev/null || true
        cp /vagrant/.vagrant/machines/vm-tools/virtualbox/private_key /home/vagrant/keys/vm-tools_key 2>/dev/null || true
        chmod 600 /home/vagrant/keys/* 2>/dev/null || true
        chown vagrant:vagrant /home/vagrant/keys/* 2>/dev/null || true
      SHELL

      # Triggers when k8s-worker-2 is up = all K8s nodes exist
      if node[:name] == "k8s-worker-2"
        node_config.vm.provision "ansible_local" do |ansible|
          ansible.playbook       = "ansible/playbooks/k8s.yml"
          ansible.inventory_path = "ansible/inventory/hosts.ini"
          ansible.config_file    = "ansible/ansible.cfg"
          ansible.limit          = "all"
          ansible.verbose        = "v"
          ansible.compatibility_mode = "2.0"
        end
      end

      # Triggers when vm-tools is up = independent from K8s
      if node[:name] == "vm-tools"
        node_config.vm.provision "ansible_local" do |ansible|
          ansible.playbook       = "ansible/playbooks/vm-tools.yml"
          ansible.inventory_path = "ansible/inventory/hosts.ini"
          ansible.config_file    = "ansible/ansible.cfg"
          ansible.limit          = "all"
          ansible.verbose        = "v"
          ansible.compatibility_mode = "2.0"
        end
      end
    end
  end
end