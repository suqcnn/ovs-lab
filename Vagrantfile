# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2"

# NOTE: Netmask is assumed 255.255.255.0 for all

SERVER1_TUNNEL_IP      = "192.168.1.10"
SERVER1_TUNNEL_GATEWAY = "192.168.1.254"
SERVER1_MGMT_IP        = "192.168.101.10"

SERVER2_TUNNEL_IP      = "192.168.2.20"
SERVER2_TUNNEL_GATEWAY = "192.168.2.254"
SERVER2_MGMT_IP        = "192.168.101.20"

INTERNET_MGMT_IP       = "192.168.101.254"


Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "ubuntu/trusty64"


  # "INTERNET"
  config.vm.define "internet" do |server|
    server.vm.hostname = "internet"

    # Server 1 tunnel transport network
    server.vm.network "private_network",
                      virtualbox__intnet: "server1_net",
                      ip: SERVER1_TUNNEL_GATEWAY,
                      netmask: "255.255.255.0"

    # Server 2 tunnel transport network
    server.vm.network "private_network",
                      virtualbox__intnet: "server2_net",
                      ip: SERVER2_TUNNEL_GATEWAY,
                      netmask: "255.255.255.0"

    # Management/Control network
    server.vm.network "private_network",
                      ip: INTERNET_MGMT_IP,
                      netmask: "255.255.255.0"

    server.vm.provider "virtualbox" do |vb|
      vb.customize ["modifyvm", :id, "--memory", 256]
      vb.customize ["setextradata", :id, "VBoxInternal2/SharedFoldersEnableSymlinksCreate/v-root", "1"]
    end

    server.vm.provision "shell", inline: <<-SCRIPT
      [ "$(sysctl --values net.ipv4.ip_forward)" -eq "1" ] || {
        # Persist across reboots
        echo "net.ipv4.ip_forward=1" >> /etc/sysctl.conf # enable ip4 forwarding
        sysctl -p # apply settings from /etc/sysctl.conf
      }
    SCRIPT

    server.vm.provision "puppet" do |puppet|
      puppet.manifests_path = "puppet/manifests"
      puppet.manifest_file  = "site.pp"
      puppet.options = "--verbose --debug"
    end
  end


  # SERVER 1
  config.vm.define "server1" do |server|
    server.vm.hostname = "server1"

    # Tunnel transport network
    server.vm.network "private_network",
                      virtualbox__intnet: "server1_net",
                      ip: SERVER1_TUNNEL_IP,
                      netmask: "255.255.255.0",
                      auto_config: false

    # Provision tunnel transport interface via file, so we can
    # add a persistent static route to tunnel gateway
    server.vm.provision "shell", inline: <<-SCRIPT
      [ -e /etc/network/interfaces.d/eth1.cfg ] || {
         cat << EOT >/etc/network/interfaces.d/eth1.cfg
auto eth1
iface eth1 inet static
      address #{SERVER1_TUNNEL_IP}
      netmask 255.255.255.0
      up ip route add #{SERVER2_TUNNEL_IP}/32 via #{SERVER1_TUNNEL_GATEWAY} dev eth1

EOT
         ifup eth1
      }
    SCRIPT

    # Management/Control network
    server.vm.network "private_network",
                      ip: SERVER1_MGMT_IP,
                      netmask: "255.255.255.0"

    server.vm.provider "virtualbox" do |vb|
      vb.customize ["modifyvm", :id, "--memory", 256]
      vb.customize ["setextradata", :id, "VBoxInternal2/SharedFoldersEnableSymlinksCreate/v-root", "1"]
    end

    server.vm.provision "puppet" do |puppet|
      puppet.manifests_path = "puppet/manifests"
      puppet.manifest_file  = "site.pp"
      puppet.options = "--verbose --debug"
    end
  end


  # SERVER 2
  config.vm.define "server2" do |server|
    server.vm.hostname = "server2"

    # Tunnel transport network
    server.vm.network "private_network",
                      virtualbox__intnet: "server2_net",
                      ip: SERVER2_TUNNEL_IP,
                      netmask: "255.255.255.0",
                      auto_config: false

    # Provision tunnel transport interface via file, so we can
    # add a persistent static route to tunnel gateway
    server.vm.provision "shell", inline: <<-SCRIPT
      [ -e /etc/network/interfaces.d/eth1.cfg ] || {
         cat << EOT >/etc/network/interfaces.d/eth1.cfg
auto eth1
iface eth1 inet static
      address #{SERVER2_TUNNEL_IP}
      netmask 255.255.255.0
      up ip route add #{SERVER1_TUNNEL_IP}/32 via #{SERVER2_TUNNEL_GATEWAY} dev eth1

EOT
         ifup eth1
      }
    SCRIPT

    # Management/Control network
    server.vm.network "private_network",
                      ip: SERVER2_MGMT_IP,
                      netmask: "255.255.255.0"

    server.vm.provider "virtualbox" do |vb|
      vb.customize ["modifyvm", :id, "--memory", 256]
      vb.customize ["setextradata", :id, "VBoxInternal2/SharedFoldersEnableSymlinksCreate/v-root", "1"]
    end

    server.vm.provision "puppet" do |puppet|
      puppet.manifests_path = "puppet/manifests"
      puppet.manifest_file  = "site.pp"
      puppet.options = "--verbose --debug"
    end
  end


end
