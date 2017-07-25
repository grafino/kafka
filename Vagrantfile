# -*- mode: ruby -*-
# vim: set ft=ruby:

Vagrant.configure("2") do |config|

  config.vm.box = "bento/centos-7.3"
  config.vm.synced_folder ".", "/vagrant"
  config.ssh.forward_agent = true

  config.vm.provider "virtualbox" do |vb|
    vb.customize [
      "modifyvm", :id,
      "--memory", "2048",
      "--cpus", "1",
      "--ioapic", "on",
      "--pae", "on",
      "--hwvirtex", "on",
      "--vtxvpid", "on",
      "--vtxux", "on",
      "--nestedpaging", "off"
    ]
  end

  config.vm.define "broker01" do |broker01|
    broker01.vm.hostname = "broker01.kafka.local"
    broker01.vm.network "private_network", ip: "192.168.50.21"

    broker01.vm.provider :virtualbox do |vb|
      vb.customize ["modifyvm", :id, "--name", "kafka-broker01"]
    end

  end

  config.vm.define "broker02" do |broker02|
    broker02.vm.hostname = "broker02.kafka.local"
    broker02.vm.network "private_network", ip: "192.168.50.22"

    broker02.vm.provider :virtualbox do |vb|
      vb.customize ["modifyvm", :id, "--name", "kafka-broker02"]
    end

  end

  config.vm.define "broker03" do |broker03|
    broker03.vm.hostname = "broker03.kafka.local"
    broker03.vm.network "private_network", ip: "192.168.50.23"

    broker03.vm.provider :virtualbox do |vb|
      vb.customize ["modifyvm", :id, "--name", "kafka-broker03"]
    end

  end

end
