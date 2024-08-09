# -*- mode: ruby -*-
# vi: set ft=ruby :

require 'yaml'

cfg = YAML.safe_load(File.read('lab-config.yml'))

# Check no more than 3 masters requested
if cfg['master_nodes'] > 3
  puts('ERROR: this lab does not support more than 3 master nodes.')
  exit(1)
end

# Check no more than 9 workers requested
if cfg['worker_nodes'] > 9
  puts('ERROR: this lab does not support more than 9 worker nodes.')
  exit(1)
end

private_network_range = "192.168.56.0/24"

Vagrant.configure("2") do |config|

  config.vm.provider "virtualbox" do |vbox|
    vbox.gui = false
  end

  # Set VM image
  config.vm.box = "debian/bookworm64"
  config.vm.box_check_update = false

  # Create masters
  (1..cfg['master_nodes']).each do |node_id|
    config.vm.define "master-#{node_id}" do |node|
      node.vm.hostname = "master-#{node_id}"
      node.vm.network "private_network",
        :name => cfg['host_network_name'],
        :adapter => 2,
        :ip => "192.168.56.#{cfg['host_network_master_range'] + node_id}"
    end
  end

  # Create workers
  (1..cfg['worker_nodes']).each do |node_id|
    config.vm.define "worker-#{node_id}" do |node|
      node.vm.hostname = "master-#{node_id}"
      node.vm.network "private_network",
        :name => cfg['host_network_name'],
        :adapter => 2,
        :ip => "192.168.56.#{cfg['host_network_worker_range'] + node_id}"
    end
  end

end
