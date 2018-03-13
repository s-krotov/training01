# -*- mode: ruby -*-
# vi: set ft=ruby :
Vagrant.configure("2") do |config|
  config.vm.box = "bertvv/centos72"
  #config.vm.network "public_network", bridge: "Default Switch"
  config.vm.provider "virtualbox" do |h|
    h.memory = 4096
    #h.enable_virtualization_extensions = true
    #h.differencing_disk = true
  end
  node1 = 'grafana'
  config.vm.define "#{node1}" do |n|
    n.vm.hostname = "#{node1}"

    n.vm.network "forwarded_port", guest: 3000, host: 3000
    #config.vm.synced_folder ".", "/vagrant", type: "nfs", create: true
    #config.vm.provision :shell, :inline => "echo '#{id_rsa_ssh_key_pub }' >> /home/vagrant/.ssh/authorized_keys"
    n.vm.provision "shell", inline: <<-SHELL
      systemctl stop firewalld
      systemctl disable firewalld

      yum install yum-utils -y  
      yum install epel-release -y
      yum install collectd -y
      cat /vagrant/collectd.conf > /etc/collectd.conf
      systemctl enable collectd
      systemctl start collectd
      #importing key or not
      isInFile=$(cat .ssh/authorized_keys | grep "mykey")
      if [ $isInFile -ne 0 ]; then
        then
          echo "Key already present"
        else
          ssh-keygen -C "mykey" -q -N "" -f /home/vagrant/id_rsa
          cat id_rsa.pub >> /home/vagrant/.ssh/authorized_keys
          cp id_rsa /vagrant/id_rsa_$hostname
      fi
      #docker
      yum-config-manager \ --add-repo \ https://download.docker.com/linux/centos/docker-ce.repo
      yum install docker-ce -y
      systemctl enable docker
      systemctl start docker
      #grafana
      docker run -d --name=grafana -p 3000:3000 grafana/grafana
      #influx
      cat /vagrant/influxdb.repo > /etc/yum.repos.d/influxdb.repo
      yum install influxdb -y
      cat /vagrant/influxdb.conf > /etc/influxdb/influxdb.conf
      systemctl enable influxdb
      systemctl start influxdb
      #elastic
      sysctl -w vm.max_map_count=262144

      #docker run -p 9200:9200 -p 9300:9300 -e "" docker.elastic.co/elasticsearch/elasticsearch:6.2.2
      #docker run --name elasticsearch -p 9200:9200 -p 9300:9300 -e "xpack.security.enabled=false" docker.elastic.co/elasticsearch/elasticsearch
    SHELL
    #n.vm.provision  :shell, :inline => influx_install
  end
  node2 = 'collectd'
  config.vm.define "#{node2}" do |n|
    n.vm.provider :virtualbox do |v|
      v.customize ["modifyvm", :id, "--memory", "512"]
    end
    n.vm.hostname = "#{node2}"
    n.vm.provision "shell", inline: <<-SHELL
      systemctl stop firewalld
      systemctl disable firewalld
      yum install epel-release -y
      yum install collectd -y
      cat /vagrant/collectd1.conf > /etc/collectd.conf
      systemctl enable collectd
      systemctl start collectd
    SHELL
    isInFile=$(cat .ssh/authorized_keys | grep "mykey")
    if [ $isInFile -ne 0 ]; then
      then
        echo "Key already present"
      else
        ssh-keygen -C "mykey" -q -N "" -f /home/vagrant/id_rsa
        cat id_rsa.pub >> /home/vagrant/.ssh/authorized_keys
        cp id_rsa /vagrant/id_rsa_$hostname
    fi
  end
end