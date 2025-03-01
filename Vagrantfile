# -*- mode: ruby -*-
# vi: set ft=ruby :
ENV['VAGRANT_NO_PARALLEL'] = 'yes'
Vagrant.configure(2) do |config|

  base_ip_str = "10.240.0.1"
  number_cplane = 1 # Number of cplane nodes kubernetes
  cpu_cplane = 2
  mem_cplane = 1792
  number_worker = 1 # Number of workers nodes kubernetes
  cpu_worker = 1
  mem_worker = 1024
  config.vm.box = "generic/ubuntu2204" # Image for all installations
  kubectl_version = "1.31"
  kube_version = "1.31"

# Compute nodes
  number_machines = number_cplane + number_worker

  nodes = []
  (0..number_machines).each do |i|
    case i
      when 0
        nodes[i] = {
          "name" => "controller",
          "ip" => "#{base_ip_str}#{i}"
        }
      when 1..number_cplane
        nodes[i] = {
          "name" => "cplane#{i}",
          "ip" => "#{base_ip_str}#{i}"
        }
      when number_cplane..number_machines
        nodes[i] = {
          "name" => "worker#{i-number_cplane}",
          "ip" => "#{base_ip_str}#{i}"
        }
    end
  end

# Provision VM
  nodes.each do |node|
    config.hostmanager.enabled = true
    config.hostmanager.manage_host = true
    config.vm.define node["name"] do |machine|
      machine.vm.hostname = node["name"]
      machine.vm.provider "libvirt" do |lv|
        if (node["name"] =~ /cplane/)
          lv.cpus = cpu_cplane
          lv.memory = mem_cplane
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
          "cplanes" => ["cplane[1:#{number_cplane}]"],
          "workers" => ["worker[1:#{number_worker}]"],
          "kubernetes:children" => ["cplanes", "workers"],
          "all:vars" => {
            "base_ip_str" => "#{base_ip_str}",
            "kubectl_version" => "#{kubectl_version}",
            "kube_version" => "#{kube_version}",
            "number_cplane" => "#{number_cplane}"
          }
        }
      end
    end
  end
  config.push.define "local-exec" do |push|
    push.inline = <<-SCRIPT
      ansible-playbook -i .vagrant/provisioners/ansible/inventory/vagrant_ansible_inventory playbooks/init-cluster.yml -u vagrant
    SCRIPT
  end
end
