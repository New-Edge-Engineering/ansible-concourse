# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/xenial64"
  config.vm.define "vagrantci" do |vagrantci|
  end

  config.vm.network "private_network", ip: "172.16.50.150"

  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "test/vagrant.yml"
#    ansible.verbose = "vvvv"
    ansible.groups = {
      "concourse-web" => ["vagrantci"],
      "concourse-worker" => ["vagrantci"],
      }
  end
end
