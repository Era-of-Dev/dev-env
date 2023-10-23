# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.

require 'getoptlong'

opts = GetoptLong.new(
  # common vagrant flags
  [ '-f', GetoptLong::OPTIONAL_ARGUMENT ],
  [ '-c', GetoptLong::OPTIONAL_ARGUMENT ],
  [ '--provision', GetoptLong::OPTIONAL_ARGUMENT ],
  [ '--provider', GetoptLong::OPTIONAL_ARGUMENT ],
  [ '--all', GetoptLong::OPTIONAL_ARGUMENT ],
  # Custom flags
  [ '--os', GetoptLong::OPTIONAL_ARGUMENT ],
  [ '--verbose', GetoptLong::OPTIONAL_ARGUMENT ],
  [ '--db', GetoptLong::OPTIONAL_ARGUMENT ]
)

# Default variables
operatingSystem='bento/rockylinux-9'
optsDebug=false
optsVerbosity=''

opts.each do |opt, arg|
  case opt
    when '--os'
      operatingSystem=arg
    when '--verbose'
      optsVerbosity='-vv'
    when '--db'
      optsDebug=true
  end
end

Vagrant.configure("2") do |config|
    # The most common configuration options are documented and commented below.
    # For a complete reference, please see the online documentation at
    # https://docs.vagrantup.com.
  
  
      # Set bridged network device to allow VM connectivity to other physical machines on the network
      # NOTE: The bridge network device name is specific to your machine and must be changed to
      #       select the desired network device.
      BRIDGE_NETWORK_ENABLE=false

  ###############################################################################
  # Create and configure dev Server
  config.vm.define "dev" do |dev|
    dev.vm.box = operatingSystem
    dev.vm.hostname = "dev"

    dev.vm.provider "virtualbox" do |vb|
      # Customize the amount of memory on the VM:
      vb.memory = "2048"
    end

    # Set the IP address configuration for the VM
    vm_ip_address = "192.168.56.111"
    vm_netmask = "255.255.255.0"

    # Disable synced folder to speed up boot time due to large ISO files in ansible provisoining folder
    dev.vm.synced_folder ".", "/vagrant", disabled: true

    # Forward ports 9090 Cockpit for dev Server to be seen by the client
    # dev.vm.network "forwarded_port", guest: 9090, host: 9090

    # Select network configuration based on BRIDGE_NETWORK_ENABLE above.
    if (BRIDGE_NETWORK_ENABLE==true)
      dev.vm.network "public_network",ip: vm_ip_address, netmask: vm_netmask, bridge: BRIDGE_NETWORK_DEVICE_NAME
    else
      dev.vm.network "private_network", ip: vm_ip_address, netmask: vm_netmask
    end

    # Run ansible provisioner against the "dev" node
    dev.vm.provision "ansible" do |ansible|
      ansible.playbook = "site.yml"
      ansible.config_file = "ansible.cfg"
      ansible.inventory_path = "hosts.yml"
      ansible.limit = "dev"
      ansible.compatibility_mode = "2.0"
      ansible.verbose = optsVerbosity
      ansible.become = true
      ansible.extra_vars = {
        debug: optsDebug
      }
      if (ENV.has_key?('NVIDIA'))
        ansible.tags = "nvidia"
      end
    end
  end
end
  