# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|

  base_ip_str = "10.240.0.1"
  number_master = 1 # Number of master nodes kubernetes
  cpu_master = 2
  mem_master = 1792
  number_worker = 2 # Number of workers nodes kubernetes
  cpu_worker = 1
  mem_worker = 1024
  config.vm.box = "generic/ubuntu2004" # Image for all installations
  kubectl_version = "1.23.1-00"
  kube_version = "1.23.1-00"
  docker_version = "5:20.10.12~3-0~ubuntu-focal"


# Compute nodes
  number_machines = number_master + number_worker

  nodes = []
  (0..number_machines).each do |i|
    case i
      when 0
        nodes[i] = {
          "name" => "controller",
          "ip" => "#{base_ip_str}#{i}"
        }
      when 1..number_master
        nodes[i] = {
          "name" => "master#{i}",
          "ip" => "#{base_ip_str}#{i}"
        }
      when number_master..number_machines
        nodes[i] = {
          "name" => "worker#{i-number_master}",
          "ip" => "#{base_ip_str}#{i}"
        }
    end
  end

# Provision VM
  nodes.each do |node|
    config.vm.define node["name"] do |machine|
      machine.vm.hostname = node["name"]
      machine.vm.provider "libvirt" do |lv|
        if (node["name"] =~ /master/)
          lv.cpus = cpu_master
          lv.memory = mem_master
        else
          lv.cpus = cpu_worker
          lv.memory = mem_worker
        end
      end
      machine.vm.synced_folder '.', '/vagrant', disabled: true
      machine.vm.network "private_network", ip: node["ip"]
      machine.vm.provision "ansible" do |ansible|
        ansible.playbook = "playbooks/provision.yml"
        ansible.groups = {
          "masters" => ["master[1:#{number_master}]"],
          "workers" => ["worker[1:#{number_worker}]"],
          "kubernetes:children" => ["masters", "workers"],
          "all:vars" => {
            "base_ip_str" => "#{base_ip_str}",
            "kubectl_version" => "#{kubectl_version}",
            "kube_version" => "#{kube_version}",
            "docker_version" => "#{docker_version}",
            "number_master" => "#{number_master}"
          }
        }
      end
    end
  end
end
