# Create a VM running Ubuntu 22.04 LTS with ansible as provisioner.

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/jammy64"
  config.vm.hostname = "sysops-poc"

  # Private network for security
  config.vm.network "private_network", type: "dhcp"
  config.vm.network "forwarded_port", guest: 80, host: 8888


  # Resource allocation
  config.vm.provider "virtualbox" do |vb|
    vb.memory = 2048 # 2GB RAM
    vb.cpus = 2      # 2 CPUs
  end

  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "ansible/playbook.yaml"
  end
end
