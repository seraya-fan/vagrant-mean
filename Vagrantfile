# -*- mode: ruby -*-
# vi: set ft=ruby :


# Defaults
PREFS = {
  :domain => "localhost",
  :app_server_name => "app",
  :app_server_ip => "192.168.33.101",
  :db_server_ip => "192.168.33.102",
  :app_box => "ubuntu_saucy64",
  :app_share => "./code",
  :app_box_url => "http://cloud-images.ubuntu.com/vagrant/saucy/current/saucy-server-cloudimg-amd64-vagrant-disk1.box",
}

# Include the ovverrides if it is set
if File.exist?("prefs.rb")
  require_relative "prefs.rb"
end

unless Vagrant.has_plugin?("vagrant-vbguest")
   raise 'Vagrant-vbguest is not installed! Please run `vagrant plugin install vagrant-vbguest` before continuing`.'
end

Vagrant.configure("2") do |config|

  config.vm.define "app", primary: true do |app|

    app.vm.hostname = PREFS[:app_server_name] + "." + PREFS[:domain]

    # Every Vagrant virtual environment requires a box to build off of.
    app.vm.box = PREFS[:app_box]
    # The url from where the 'config.vm.box' box will be fetched if it
    # doesn't already exist on the user's system.
    app.vm.box_url = PREFS[:app_box_url]

    # Create a forwarded port mapping which allows access to a specific port
    # within the machine from a port on the host machine. 
    # Mean.io uses a default of 3000
    app.vm.network :forwarded_port, guest: 3000, host: 3000

    # Create a private network, which allows host-only access to the machine
    # using a specific IP.
    app.vm.network :private_network, ip: PREFS[:app_server_ip]

    # Create a public network, which generally matched to bridged network.
    # Bridged networks make the machine appear as another physical device on
    # your network.
    # app.vm.network :public_network

    # If true, then any SSH connections made will enable agent forwarding.
    # Default value: false
    # app.ssh.forward_agent = true

    # Share an additional folder to the guest VM. The first argument is
    # the path on the host to the actual folder. The second argument is
    # the path on the guest to mount the folder. And the optional third
    # argument is a set of non-required options.
    app.vm.synced_folder PREFS[:app_share], "/mnt/code", :nfs => PREFS[:nfs]

    # Provider-specific configuration so you can fine-tune various
    # backing providers for Vagrant. These expose provider-specific options.
    app.vm.provider :virtualbox do |vb|
      # Set the name; this is used in VirtualBox so that it's easy to parse what box is running, etc.
      vb.name = 'apptestname' #PREFS[:app_server_name] + "_" + Time.now.strftime('%s')
      vb.customize ["modifyvm", :id, "--memory", "1024"]
      vb.customize ["modifyvm", :id, "--cpus", "4"]
      vb.customize ["modifyvm", :id, "--ioapic", "on"]
      
      # force the box to proxy the DNS requests through the NAT;
      # this should cause less network issues when DNS is changing while wifi/vpn drops randomly
      # http://www.virtualbox.org/manual/ch09.html#nat-adv-dns
      vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
    end

    app.vm.provision "ansible" do |ansible|
      ansible.inventory_path = "files/hosts"
      ansible.playbook = "tasks/main.yml"
      ansible.verbose = 'vvvv'
      ansible.limit = 'appserver'
      # For development purposes
      ansible.host_key_checking = false
    end
  end


  config.vm.define "db", primary: true do |db|

    db.vm.hostname = PREFS[:app_server_name] + "db." + PREFS[:domain]

    # Every Vagrant virtual environment requires a box to build off of.
    db.vm.box = PREFS[:app_box]
    # The url from where the 'config.vm.box' box will be fetched if it
    # doesn't already exist on the user's system.
    db.vm.box_url = PREFS[:app_box_url]

    # Mongo default
    db.vm.network :forwarded_port, guest: 27017, host: 27017
    # Mongo status default
    db.vm.network :forwarded_port, guest: 28017, host: 28017

    # Create a private network, which allows host-only access to the machine
    # using a specific IP.
    db.vm.network :private_network, ip: PREFS[:db_server_ip]

    # Provider-specific configuration so you can fine-tune various
    # backing providers for Vagrant. These expose provider-specific options.
    db.vm.provider :virtualbox do |vb|
      # Set the name; this is used in VirtualBox so that it's easy to parse what box is running, etc.
      vb.name = PREFS[:app_server_name] + "db_" + Time.now.strftime('%s')
      vb.customize ["modifyvm", :id, "--memory", "256"]
      vb.customize ["modifyvm", :id, "--cpus", "2"]
      vb.customize ["modifyvm", :id, "--ioapic", "on"]

      
      # force the box to proxy the DNS requests through the NAT;
      # this should cause less network issues when DNS is changing while wifi/vpn drops randomly
      # http://www.virtualbox.org/manual/ch09.html#nat-adv-dns
      vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
    end

    db.vm.provision "ansible" do |ansible|
      ansible.inventory_path = "files/hosts"
      ansible.playbook = "tasks/db.yml"
      ansible.limit = 'appdbserver'
      ansible.verbose = 'vvvv'
      # For development purposes
      ansible.host_key_checking = false
    end
  end

end
