# -*- mode: ruby -*-
# vi: set ft=ruby :

VM_COUNT = 3
LB_HOSTNAME = 'lb'

node_script = <<SCRIPT
  #!/bin/bash
  #install 
  yum install java-1.8.0-openjdk tomcat tomcat-webapps tomcat-admin-webapps -y
  
  #create sample data
  mkdir /usr/share/tomcat/webapps/test
  echo "<h1>i am ${1}</h1>" > /usr/share/tomcat/webapps/test/index.html
  
  echo "worker.lb.balance_workers=${1}" > /vagrant/worker_${1}.tmp
  echo "worker.${1}.host=${2}" >> /vagrant/worker_${1}.tmp
  echo "worker.${1}.port=8009" >> /vagrant/worker_${1}.tmp
  echo "worker.${1}.type=ajp13" >> /vagrant/worker_${1}.tmp
  
  #starting node services
  systemctl enable tomcat
  systemctl start tomcat
  
  #TODO firewall setup vs disable
  systemctl stop firewalld
  systemctl disable firewalld
SCRIPT

lb_script = <<SCRIPT
  #!/bin/bash
  yum install httpd java-1.8.0-openjdk-devel -y
  wget -O /etc/yum.repos.d/jenkins.repo http://pkg.jenkins-ci.org/redhat/jenkins.repo
  rpm --import https://jenkins-ci.org/redhat/jenkins-ci.org.key
  yum install jenkins -y
  systemctl enable jenkins.service
  systemctl start jenkins.service

  mkdir /app && cd /app
  wget -c -N -q https://sonatype-download.global.ssl.fastly.net/nexus/oss/nexus-2.14.7-01-bundle.tar.gz
  tar xvf nexus-2.14.7-01-bundle.tar.gz -C /app
  ln -s /app/nexus-2.14.7-01/ /app/nexus
  adduser nexus
  chown -R nexus:nexus /app/nexus-2.14.7-01
  chown -R nexus:nexus /app/sonatype-work
  export NEXUS_HOME=/app/nexus 
  echo 'RUN_AS_USER="nexus"' > /app/nexus/bin/nexus
  ln -s /opt/nexus/bin/nexus /etc/init.d/nexus
  systemctl enable nexus
  systemctl start nexus
  #8081/nexus
  
  cp /vagrant/mod_jk.so /etc/httpd/modules/
  cp /vagrant/lb.conf /etc/httpd/conf.d/
  
  cp /vagrant/workers.template /etc/httpd/conf/workers.properties
  cat /vagrant/worker_*.tmp >> /etc/httpd/conf/workers.properties
  #rm /vagrant/worker*.tmp -f
  
  systemctl enable httpd
  systemctl start httpd
  systemctl restart httpd
  
  systemctl enable jenkins
  systemctl start jenkins

  #TODO:create rules
  systemctl stop firewalld
  systemctl disable firewalld
SCRIPT



Vagrant.configure("2") do |config|
  config.vm.box = "bertvv/centos72"
  config.vm.provider "virtualbox" do |vb|
    #vb.gui = true
    vb.memory = 1024
  end

  (VM_COUNT-1).times do |i|
    node_id = 'vm' + (i+1).to_s
    node_ip = '192.168.56.' + (11 + i).to_s
    config.vm.define node_id do |node|
      node.vm.hostname = "#{node_id}"
      node.vm.network :private_network, ip: "#{node_ip}", name: "VirtualBox Host-Only Ethernet Adapter"
      node.vm.provision :shell, :inline => node_script, :args => "'#{node_id}' '#{node_ip}'"
    end
  end

  config.vm.define LB_HOSTNAME do |node|
    node.vm.hostname = LB_HOSTNAME
    node.vm.network :private_network, ip: "192.168.56.10", name: "VirtualBox Host-Only Ethernet Adapter" #name parameter is due https://github.com/hashicorp/vagrant/issues/6059#issuecomment-126834560
    node.vm.network "forwarded_port", guest: 80, host: 8000
    node.vm.network "forwarded_port", guest: 8080, host: 8080
    node.vm.network "forwarded_port", guest: 8081, host: 8081
    node.vm.provision :shell, :inline => lb_script
  end
end