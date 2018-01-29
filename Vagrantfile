# -*- mode: ruby -*-
# vi: set ft=ruby :
unless Vagrant.has_plugin?("vagrant-hosts")
  raise "Please, run 'vagrant plugin install vagrant-hosts' before running 'vagrant up'"
end
Vagrant.configure("2") do |config|
  config.vm.box = "bertvv/centos72"
  # config.vm.box_check_update = false
  # config.vm.network "forwarded_port", guest: 80, host: 8080
  # config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  config.vm.provider "virtualbox" do |vb|
    # Display the VirtualBox GUI when booting the machine
    vb.gui = true
    # Customize the amount of memory on the VM:
    vb.memory = "512"
  end

  ssh_pub_key = File.readlines("id_rsa.pub").first.strip
  config.vm.provision "file", source: "./id_rsa", destination: "~/.ssh/id_rsa"
  config.vm.provision 'shell', inline: 'mkdir -p /root/.ssh'
  config.vm.provision 'shell', inline: "echo #{ssh_pub_key} >> /root/.ssh/authorized_keys"
  config.vm.provision 'shell', inline: "echo #{ssh_pub_key} >> /home/vagrant/.ssh/authorized_keys", privileged: false


  config.vm.define :vm1 do |node|
    node.vm.hostname = "vm1"
    node.vm.network :private_network, :ip => '192.168.56.56'
    node.vm.provision "shell", inline: <<-SHELL
      yum install git -y
      git clone https://github.com/s-krotov/module2.git -b task2
      cat module2/README.md
      chmod 0600 .ssh/id_rsa
      grep -qF "192.168.56.66 vm2" "/etc/hosts" || echo "192.168.56.66 slave vm2" >> "/etc/hosts"
    SHELL
  end

  config.vm.define :vm2 do |node|
    node.vm.hostname = "vm2"
    node.vm.network :private_network, :ip => '192.168.56.66'
    node.vm.provision "shell", inline: <<-SHELL
      chmod 0600 .ssh/id_rsa
      grep -qF "192.168.56.56 vm1" "/etc/hosts" || echo "192.168.56.56 master vm1" >> "/etc/hosts"
    SHELL
  end

end
