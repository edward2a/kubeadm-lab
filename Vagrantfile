# -*- mode: ruby -*-
# vi: set ft=ruby :

require 'yaml'

cfg = YAML.safe_load(File.read('lab-config.yml'))
cfg['ssh_public_key'] = File.read(cfg['ssh_pub_key_path']).strip()

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

# Create ansible inventory
ans_inventory = {
  "all" => {
    "vars" => {
      "kubernetes_cni_base_version" => "1.2",
      "kubernetes_major_version" => "1",
      "kubernetes_minor_version" => "26",
      "kubernetes_patch_version" => "12",
      "kubernetes_version" => '{{ kubernetes_major_version }}.{{ kubernetes_minor_version }}.{{ kubernetes_patch_version }}',
      "weavenet_version" => "2.8.1",
      "weavenet_dl_url" => "https://github.com/weaveworks/weave/releases/download/v{{ weavenet_version }}/weave-daemonset-k8s.yaml"
    },
    "hosts" => {},
    "children" => {
      "nfs_server" => {
        "hosts" => {},
      },
      "k8s_all" => {
        "hosts" => {},
      },
      "k8s_masters" => {
        "hosts" => {},
      },
      "k8s_init_master" => {
        "hosts" => {},
      },
      "k8s_workers" => {
        "hosts" => {},
      },
    },
  }
}

Vagrant.configure("2") do |config|

  config.vm.provider "virtualbox" do |vbox|
    vbox.gui = false
  end

  # Set VM image
  config.vm.box = "debian/bookworm64"
  config.vm.box_check_update = false

  # Create NFS server
  # TODO

  # Create masters
  (1..cfg['master_nodes']).each do |node_id|
    config.vm.define "master-#{node_id}" do |node|
      node.vm.hostname = "master-#{node_id}"
      node.vm.network "private_network",
        :name => cfg['host_network_name'],
        :adapter => 2,
        :ip => "#{cfg['host_network_prefix']}.#{cfg['host_network_master_range'] + node_id}"

      # Configure VM resources
      node.vm.provider "virtualbox" do |vm|
        vm.memory = cfg["master_config"]["memory"]
        vm.cpus = cfg["master_config"]["cpu"]
      end
    end

    # Add node to inventory
    ans_inventory['all']['hosts']["master-#{node_id}"] = {
      "ansible_host" => "#{cfg['host_network_prefix']}.#{cfg['host_network_master_range'] + node_id}",
      "node_type" => "k8s-master",
    }

    ans_inventory['all']['children']['k8s_masters']['hosts']["master-#{node_id}"] = {}
    ans_inventory['all']['children']['k8s_all']['hosts']["master-#{node_id}"] = {}

    if node_id == 1
      ans_inventory['all']['children']['k8s_init_master']['hosts']["master-#{node_id}"] = {}
    end

  end

  # Create workers
  (1..cfg['worker_nodes']).each do |node_id|
    config.vm.define "worker-#{node_id}" do |node|
      node.vm.hostname = "worker-#{node_id}"
      node.vm.network "private_network",
        :name => cfg['host_network_name'],
        :adapter => 2,
        :ip => "#{cfg['host_network_prefix']}.#{cfg['host_network_worker_range'] + node_id}"

      # Configure VM resources
      node.vm.provider "virtualbox" do |vm|
        vm.memory = cfg["worker_config"]["memory"]
        vm.cpus = cfg["worker_config"]["cpu"]
      end
    end

    # Add node to inventory
    ans_inventory['all']['hosts']["worker-#{node_id}"] = {
      "ansible_host" => "#{cfg['host_network_prefix']}.#{cfg['host_network_worker_range'] + node_id}",
      "node_type" => "k8s-worker",
    }

    ans_inventory['all']['children']['k8s_workers']['hosts']["worker-#{node_id}"] = {}
    ans_inventory['all']['children']['k8s_all']['hosts']["worker-#{node_id}"] = {}

  end

  # Add SSH public key(s) to node
  config.vm.provision "shell" do |s|
    s.inline = <<-SHELL
      echo "#{cfg['ssh_public_key']}" >> /home/vagrant/.ssh/authorized_keys
      echo "#{cfg['ssh_public_key']}" >> /root/.ssh/authorized_keys
    SHELL
  end

  # Write ansible inventory
  File.write('inventory-vbox.yml', YAML.safe_dump(ans_inventory))
end

