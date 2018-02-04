# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  config.vm.box = "bertvv/centos72"
  config.vm.provider "virtualbox" do |vb|
    vb.gui = true
    vb.memory = 1024
  end

config.vm.define :lb do |node|
  node.vm.hostname = "lb"
  #name parameter is due https://github.com/hashicorp/vagrant/issues/6059#issuecomment-126834560
  node.vm.network :private_network, ip: "192.168.56.10", name: "VirtualBox Host-Only Ethernet Adapter"
  node.vm.network "forwarded_port", guest: 80, host: 8080
  node.vm.provision "shell", inline: <<-SHELL
    yum install httpd -y
    systemctl enable httpd
    systemctl start httpd
    cp /vagrant/mod_jk.so /etc/httpd/modules/
    cp /vagrant/workers.properties /etc/httpd/conf/
    cp /vagrant/lb.conf /etc/httpd/conf.d/
    systemctl restart httpd

    #TODO:create rules
    systemctl stop firewalld
    systemctl disable firewalld
  SHELL
end

config.vm.define :vm1 do |node|
  node.vm.hostname = "vm1"
  node.vm.network :private_network, ip: "192.168.56.20", name: "VirtualBox Host-Only Ethernet Adapter"
  node.vm.provision "shell", inline: <<-SHELL
     yum install -y java-1.8.0-openjdk tomcat tomcat-webapps tomcat-admin-webapps
     systemctl enable tomcat
     systemctl start tomcat

     mkdir /usr/share/tomcat/webapps/test
     echo "<h1>i am vm1</h1>" > /usr/share/tomcat/webapps/test/index.html

     #TODO create rules
     systemctl stop firewalld
     systemctl disable firewalld
  SHELL
end

config.vm.define "vm2" do |node|
  node.vm.hostname = "vm2"
  node.vm.network :private_network, ip: "192.168.56.30", name: "VirtualBox Host-Only Ethernet Adapter"
  node.vm.provision "shell", inline: <<-SHELL
     yum install -y java-1.8.0-openjdk tomcat tomcat-webapps tomcat-admin-webapps
     systemctl enable tomcat
     systemctl start tomcat

     mkdir /usr/share/tomcat/webapps/test
     echo "<h1>i am vm2</h1>" > /usr/share/tomcat/webapps/test/index.html

     #TODO create rules
     systemctl stop firewalld
     systemctl disable firewalld
  SHELL
end

end
