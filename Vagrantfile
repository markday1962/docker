# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.synced_folder "./config", "/vargant"
  config.vm.box = "resmas/dockerlab"
  config.vm.provider "virtualbox" do |v|
    v.memory = 1024
  end
  config.vm.define "swarm-master" do |d|
    d.vm.provision :shell, path: "bootstrap.sh"
    d.vm.hostname = "swarm-master"
    d.vm.network "private_network", ip: "10.100.192.200"
    d.vm.network "forwarded_port", guest: 8080, host: 8080
  end
  config.vm.define "swarm-worker01" do |d|
    d.vm.provision :shell, path: "bootstrap.sh"
    d.vm.hostname = "swarm-worker01"
    d.vm.network "private_network", ip: "10.100.192.201"
  end
  config.vm.define "swarm-worker02" do |d|
    d.vm.provision :shell, path: "bootstrap.sh"
    d.vm.hostname = "swarm-worker02"
    d.vm.network "private_network", ip: "10.100.192.202"
  end
  config.vm.define "swarm-worker03" do |d|
    d.vm.provision :shell, path: "bootstrap.sh"
    d.vm.hostname = "swarm-worker03"
    d.vm.network "private_network", ip: "10.100.192.203"
  end
  config.vm.define "swarm-worker04" do |d|
    d.vm.provision :shell, path: "bootstrap.sh"
    d.vm.hostname = "swarm-worker04"
    d.vm.network "private_network", ip: "10.100.192.204"
  end
end
