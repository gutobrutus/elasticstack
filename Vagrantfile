# -*- mode: ruby -*-
# vi: set ft=ruby :

#Instalação de dependências na vm via script shell
$script = <<-SCRIPT
sudo yum update -y
sudo yum install docker -y
sudo yum install git -y
sudo curl -L "https://github.com/docker/compose/releases/download/1.25.5/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
sudo systemctl start docker
sudo systemctl enable docker
SCRIPT

Vagrant.configure("2") do |config|
  config.vm.box = "centos/7"
  config.vm.box_check_update = false
  config.vm.hostname = "srvclusterelk-local"
  config.vm.network "private_network", ip: "192.168.99.10"
  config.vm.network :forwarded_port, guest: 5601, host: 5601
  config.vm.network :forwarded_port, guest: 9200, host: 9200
  config.vm.network :forwarded_port, guest: 9300, host: 9300
  config.vm.provision "shell", inline: $script

  config.vm.provider "virtualbox" do |vb|
    vb.gui = false
    vb.name = "es-cluster-vm"
    vb.memory = "4096"
    vb.cpus = "2"
  end
end
