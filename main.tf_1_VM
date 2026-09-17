terraform {
  required_version = ">= 1.0.0"
  required_providers {
    libvirt = {
      source  = "dmacvicar/libvirt"
      version = "~> 0.7.0"
    }
  }
}

provider "libvirt" {
  uri = "qemu:///system"
}

# Imagem Cloud Base (Ubuntu 22.04)
resource "libvirt_volume" "ubuntu_base" {
  name   = "ubuntu-22.04-base.qcow2"
  pool   = "default"
  source = "https://cloud-images.ubuntu.com/releases/22.04/release/ubuntu-22.04-server-cloudimg-amd64.img"
  format = "qcow2"
}

# Disco da máquina única (20 GB)
resource "libvirt_volume" "vm_disk" {
  name           = "k8s-node-disk.qcow2"
  base_volume_id = libvirt_volume.ubuntu_base.id
  pool           = "default"
  size           = 21474836480 # 20 GB
}

# Disco Cloud-Init
resource "libvirt_cloudinit_disk" "vm_init" {
  name = "k8s-node-init.iso"
  pool = "default"
  user_data = <<-EOF
#cloud-config
hostname: k8s-node
fqdn: k8s-node.k8s.local
manage_etc_hosts: true
users:
  - name: ubuntu
    sudo: ALL=(ALL) NOPASSWD:ALL
    groups: users, admin
    home: /home/ubuntu
    shell: /bin/bash
    ssh_authorized_keys:
      - ${trimspace(file(pathexpand("~/.ssh/id_rsa.pub")))}
ssh_pwauth: true
disable_root: false
EOF
}

# Rede NAT dedicada
resource "libvirt_network" "cluster_net" {
  name      = "qemu_cluster_net"
  mode      = "nat"
  domain    = "k8s.local"
  addresses = ["192.168.100.0/24"]
  autostart = true
  dhcp {
    enabled = true
  }
  dns {
    enabled = true
  }
}

# Definição da VM única
resource "libvirt_domain" "k8s_node" {
  name      = "k8s-node"
  memory    = 4096
  vcpu      = 2
  cloudinit = libvirt_cloudinit_disk.vm_init.id

  # Modo de emulação compatível
  type    = "qemu"
  machine = "pc"

  network_interface {
    network_id     = libvirt_network.cluster_net.id
    wait_for_lease = false
  }

  disk {
    volume_id = libvirt_volume.vm_disk.id
  }

  console {
    type        = "pty"
    target_port = "0"
    target_type = "serial"
  }

  graphics {
    type        = "spice"
    listen_type = "address"
    autoport    = true
  }
}

output "node_info" {
  value = {
    name = libvirt_domain.k8s_node.name
    ram  = "4096 MB"
    vcpu = 2
  }
}
