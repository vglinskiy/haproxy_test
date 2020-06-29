# -*- mode: ruby -*-
# vi: set ft=ruby :
veos_box = "arista/veos-4.23.0F"
linux_box = "ubuntu/bionic64"


Vagrant.configure("2") do |config|

  config.vm.define "haproxy" do |node1|
    #config.vm.provider :virtualbox do |vb1|
    #  vb1.name = "haproxy"
    #end
    node1.vm.box = linux_box
    #host-only network
    #node.vm.network "private_network", type: "dhcp", auto_config: false
    node1.vm.network "private_network", virtualbox__intnet: "hostA_vtepA", auto_config: false
    node1.vm.hostname = "haproxy"
    node1.vm.provision "shell", inline: <<-SHELL
      sudo ip link set enp0s8 up
      sudo ip link set enp0s8 promisc on
      #sudo dhclient enp0s8
      sudo ip addr add 192.168.99.10/24 dev enp0s8
      sudo ip route add 192.168.0.0/16 via 192.168.99.1
      sudo apt-get update
      sudo apt-get install -y curl lldpd
      #sudo apt-get -y install haproxy
      sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
      sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
      sudo apt-get update
      sudo apt-get -y install docker-ce docker-ce-cli containerd.io
      mkdir /home/vagrant/haproxy
      mkdir -p /home/vagrant/run/haproxy
      cp /vagrant/haproxy/haproxy.cfg /home/vagrant/haproxy/
      sudo docker network create -d macvlan --subnet=192.168.99.0/24 --gateway 192.168.99.1 -o parent=enp0s8 --ip-range 192.168.99.64/26 edge_net
      sudo docker run -d -v /home/vagrant/haproxy:/usr/local/etc/haproxy:ro -v /home/vagrant/run/haproxy:/run/haproxy:rw --net edge_net -p 80:80 --name my_haproxy haproxy
    SHELL
  end

  config.vm.define "nginx" do |node2|
    #config.vm.provider :virtualbox do |vb2|
    #  vb2.name = "nginx"
    #end
    node2.vm.box = linux_box
    node2.vm.network "private_network", virtualbox__intnet: "hostB_vtepB", auto_config: false
    node2.vm.hostname = "nginx"
    node2.vm.provision "shell", inline: <<-SHELL
      sudo ip link set enp0s8 up
      sudo ip addr add 192.168.101.10/24 dev enp0s8
      sudo ip route add 192.168.0.0/16 via 192.168.101.1
      sudo apt-get update
      sudo apt-get -y install nginx lldpd
      sudo cp /vagrant/nginx.conf /etc/nginx
      sudo cp /vagrant/index.html /var/www/html
      sudo systemctl restart nginx
    SHELL
  end

  config.vm.define "clientbox" do |node3|
    #config.vm.provider :virtualbox do |vb3|
    #  vb3.name = "clientbox"
    #end
    node3.vm.box = linux_box
    #host-only network
    #node3.vm.network "private_network", type: "dhcp", auto_config: false
    node3.vm.network "private_network", virtualbox__intnet: "hostA_spine", auto_config: false
    node3.vm.hostname = "clientbox"
    node3.vm.provision "shell", inline: <<-SHELL
      sudo ip link set enp0s8 up
      #sudo dhclient enp0s8
      sudo ip addr add 192.168.88.10/24 dev enp0s8
      sudo ip route add 192.168.0.0/16 via 192.168.88.1
      sudo apt-get update
      sudo apt-get install -y curl lldpd
    SHELL
  end

  config.vm.define "veos" do |veos|
    veos.vm.box = veos_box
    veos.ssh.insert_key = false
    veos.vm.boot_timeout = 180
    veos.vm.synced_folder '.', '/vagrant', disabled: true
    veos.vm.network :forwarded_port, guest: 22, host: 5522, id: 'ssh'
    veos.vm.network "private_network", auto_config: false, virtualbox__intnet: "hostA_spine"
    veos.vm.network "private_network", auto_config: false, virtualbox__intnet: "hostA_vtepA"
    veos.vm.network "private_network", auto_config: false, virtualbox__intnet: "hostB_vtepB"
    veos.vm.provider :virtualbox do |vb|
      vb.name = "veos-haproxy-router"
      vb.customize ['modifyvm',:id,'--uart1','off']
      vb.customize ['modifyvm',:id, '--uartmode1','disconnected']
    end
  end


end
