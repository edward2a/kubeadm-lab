# KubeAdm Lab
---

## Requirements

### Software
- Vagrant 2.4.1
- Virtualbox (with hostnet configured)
- Ansible (core 2.16+)
- Ansible Galaxy kubernetes.core (2.4.0+)
- Python Kubernetes (version according to selected cluster version)

## Usage

Launch the lab with `vagrant up`.

Once the lab is up and running, provision with `ansible-playbook -i inventory-vbox.yml setup.yml`.

## Configuration

### SSH Configuration

`~/.ssh/config`
```
Host 192.168.56.*
    StrictHostKeyChecking no
```

### Lab configuration

The lab infrastructure can be configured in the `lab-config.yml` file.
```yaml
# Path of a public key to install into the virtual machines (vagrant and root users)
ssh_pub_key_path: ./ssh_key.pub

# Number of master (control-plane) and worker nodes
master_nodes: 1
worker_nodes: 1

# Resource config for master and worker nodes
master_config:
  cpu: 2
  memory: 2048
worker_config:
  cpu: 2
  memory: 5144

# Name of the virtualbox host network to use
host_network_name: vboxnet0

# Host network prefix
host_network_prefix: "192.168.56"

# First IP for masters (starts on range + 1)
host_network_master_range: 220

# First IP for workers (starts on range + 1)
host_network_worker_range: 230
```
