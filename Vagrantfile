# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://atlas.hashicorp.com/search.
  config.vm.box = "ubuntu/xenial64"

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  #config.vm.network "forwarded_port", guest: 80, host: 8080
  #config.vm.network "forwarded_port", guest: 8080, host: 8080
  # mongodb port
  #config.vm.network "forwarded_port", guest: 27017, host: 27017

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"
  config.vm.network "forwarded_port", guest: 8080, host: 8080
  # for neo4j DB server
  config.vm.network "forwarded_port", guest: 7474, host: 7474
  config.vm.network "forwarded_port", guest: 7687, host: 7687

  # For Python serving examples
  config.vm.network "forwarded_port", guest: 8001, host: 8001

  # config.vm.network "private_network", type: "dhcp"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  config.vm.synced_folder "./", "/synced_folder"

  config.vm.provision "file", source: "./serve_playbooks.py", destination: "serve_playbooks.py"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  # config.vm.provider "virtualbox" do |vb|
  #   # Display the VirtualBox GUI when booting the machine
  #   # vb.gui = true

  #   # Customize the amount of memory on the VM:
  #   vb.memory = "2048"
  #   vb.cpus = "2"
  #   vb.name = "neo4j_demo"
  # end

  # The following is adapted from
  # http://stackoverflow.com/a/37335639
  config.vm.provider "virtualbox" do |vb|
    mem_ratio = 3/4
    cpu_exec_cap = 75
    host = RbConfig::CONFIG['host_os']
    # Give VM 3/4 system memory & access to all cpu cores on the host
    if host =~ /darwin/
      cpus = `sysctl -n hw.ncpu`.to_i
      # sysctl returns Bytes and we need to convert to MB
      mem = `sysctl -n hw.memsize`.to_i / 1024^2 * mem_ratio
    elsif host =~ /linux/
      cpus = `nproc`.to_i
      # meminfo shows KB and we need to convert to MB
      mem = `grep 'MemTotal' /proc/meminfo | sed -e 's/MemTotal://' -e 's/ kB//'`.to_i / 1024 * mem_ratio
    else # Windows folks
      cpus = `wmic cpu get NumberOfCores`.split("\n")[2].to_i
      mem = `wmic OS get TotalVisibleMemorySize`.split("\n")[2].to_i / 1024 * mem_ratio
    end

    if cpus == 2
      cpus = 1
    elsif cpus > 2
      cpus = cpus - 2
    end
    # puts(mem)
    # puts(mem/1024)
    # puts(mem/1024^2)
    # puts "Provisioning VM with #{cpus} CPU cores (at #{cpu_exec_cap}%) and #{mem/1024} MB RAM."

    vb.customize ["modifyvm", :id, "--memory", mem/1024]
    vb.customize ["modifyvm", :id, "--cpus", cpus]

    vb.customize ["modifyvm", :id, "--cpuexecutioncap", cpu_exec_cap]

    vb.name = "neo4j_demo"
  end



  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Define a Vagrant Push strategy for pushing to Atlas. Other push strategies
  # such as FTP and Heroku are also available. See the documentation at
  # https://docs.vagrantup.com/v2/push/atlas.html for more information.
  # config.push.define "atlas" do |push|
  #   push.app = "YOUR_ATLAS_USERNAME/YOUR_APPLICATION_NAME"
  # end

  # Enable provisioning with a shell script. Additional provisioners such as
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
  # documentation for more information about their specific syntax and use.
  config.vm.provision "shell", privileged: false, inline: <<-SHELL
    sudo apt-get update

    sudo echo "LC_ALL=\"en_US.UTF-8\"" >> /etc/environment
    sudo locale-gen UTF-8

    sudo apt-get install -y git
    sudo apt-get install -y wget

    echo "Installing Java"

    sudo apt-get -y install software-properties-common
    sudo add-apt-repository ppa:webupd8team/java
    sudo apt-get -y update
    echo oracle-java8-installer shared/accepted-oracle-license-v1-1 select true | sudo /usr/bin/debconf-set-selections
    sudo apt-get -y install oracle-java8-installer
    sudo update-java-alternatives -s java-8-oracle

    echo "Installing Neo4J"
    wget -O - https://debian.neo4j.org/neotechnology.gpg.key | sudo apt-key add -
    echo 'deb https://debian.neo4j.org/repo stable/' | sudo tee /etc/apt/sources.list.d/neo4j.list
    sudo apt-get update

    sudo apt-get install -y neo4j

    # Permission denied...
    # Do I have to add root to neo4j group?
    sudo chmod a+w /etc/neo4j/neo4j.conf
    echo "\n# Automatically added by Vagrant provision script." >> /etc/neo4j/neo4j.conf
    echo "dbms.connectors.default_listen_address=0.0.0.0" >> /etc/neo4j/neo4j.conf
    echo "dbms.memory.heap.initial_size=1024m" >> /etc/neo4j/neo4j.conf
    
    # computing the amount of memory that is available for the db server
    export memory=$(grep 'MemTotal' /proc/meminfo | sed -e 's/MemTotal://' -e 's/ kB//' | xargs)
    export db_memory=$(($memory / 1000 - 1024))
    echo "Neo4J will have $db_memory available"
    echo "dbms.memory.heap.max_size=$db_memory" >> /etc/neo4j/neo4j.conf
    echo "browser.remote_content_hostname_whitelist=*" >> /etc/neo4j/neo4j.conf

    sudo chmod a+rw -R /var/log/neo4j
    sudo chmod a+rw -R /var/lib/neo4j

    sudo neo4j restart

    sudo /usr/bin/neo4j-admin set-initial-password demo

    echo "Installing ASCIIDoctor toolchain"
    sudo apt-get install -y ruby-full
    sudo gem install asciidoctor
    sudo gem install tilt

    git clone https://github.com/neo4j-contrib/neo4j-guides

    sudo apt-get -y install python3 python3-pip python3.5-dev
    sudo pip3 install suplemon

    # cd /synced_folder/playbooks/
    # python3 serve_playbooks.py &
    cp serve_playbooks.py neo4j-guides/

    # :play http://localhost:8001/neo4j-guides/html/test.html
    # :play http://localhost:8001/html/gutenberg_locations.html

    echo "Password is: demo"
  SHELL
end

# Installation of Neo4J based on
# http://debian.neo4j.org/?_ga=1.80727511.1564702383.1486383801
# Something on MongoDB (service restart)
# https://www.linode.com/docs/databases/mongodb/install-mongodb-on-ubuntu-16-04

# https://raw.githubusercontent.com/neo4j-contrib/training/master/modeling/data/flights_initial.csv

# Installation of documentation writing toolchain:
# https://github.com/neo4j-contrib/developer-resources/blob/gh-pages/resources/guide-create-neo4j-browser-guide/guide-create-neo4j-browser-guide.adoc