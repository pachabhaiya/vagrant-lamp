# -*- mode: ruby -*-
# vi: set ft=ruby :

# Check if the 'vagrant-trigger' plugin is installed or not.
# If 'vagrant-trigger' plugin is not installed then exit.
if !Vagrant.has_plugin?('vagrant-triggers')
  puts "'vagrant-triggers' plugin not installed."
  puts "..."
  puts "Visit this link for more details:"
  puts "https://github.com/emyl/vagrant-triggers"
  exit
end

# Current vagrant directory.
vagrant_dir = File.dirname(File.expand_path(__FILE__))

# Ansible inventory file.
inventory_file = "#{vagrant_dir}/centos/playbooks/hosts"

# Include config from centos/settings.yml.
require 'yaml'
config_vm = YAML::load_file("#{vagrant_dir}/centos/settings.yml")

# Vagrant version requirements.
# Load the Vagrantfile only if the version loading it is Vagrant 1.6.3 or greater.
Vagrant.require_version ">= 1.6.3"

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  # All Vagrant configuration is done here. The most common configuration
  # options are documented and commented below. For a complete reference,
  # please see the online documentation at vagrantup.com.

  # Every Vagrant virtual environment requires a box to build off of.
  config.vm.box = config_vm['box']
  config.vm.hostname = config_vm['hostname']

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  config.vm.network "private_network", ip: config_vm['ip']

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # If true, then any SSH connections made will enable agent forwarding.
  # Default value: false
  # config.ssh.forward_agent = true

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  config.vm.synced_folder "#{vagrant_dir}/#{config_vm['synced_folder']}", "/var/www/html", type: "nfs"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  config.vm.provider "virtualbox" do |vb|
    # Don't boot with headless mode
    vb.gui = false

    # Virtual machine name
    vb.name = config_vm['hostname'] + "-vm"

    # Memory settings
    vb.memory = 1024

    # CPU settings
    vb.cpus = 2

    # The VM is modified to have a host CPU execution cap of 50%,
    # meaning that no matter how much CPU is used in the VM, no more than 50%
    # would be used on your own host machine.
    vb.customize ["modifyvm", :id, "--cpuexecutioncap", "50"]
  end

  # Enable provisioning with Ansible.
  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "#{vagrant_dir}/centos/playbooks/site.yml"
    if config_vm['ansible_verbose'] != ''
      ansible.verbose = config_vm['ansible_verbose']
    end
  end

  # Run an Ansible playbook 'vagrant-up.yml' on 'vagrant up' command.
  if !File.exist?("#{inventory_file}")
    config.trigger.before :up, :stdout => true, :force => true do
      info "Executing 'up' trigger"
      run "ansible-playbook -i #{config_vm['ip']}, --ask-sudo-pass #{vagrant_dir}/centos/playbooks/vagrant-up.yml --extra-vars \"inventory_file=#{inventory_file}\""
    end
  end

  # Run an Ansible playbook 'vagrant-halt_destroy.yml' on 'vagrant halt/destroy' command.
  if File.exist?("#{inventory_file}")
    config.trigger.before [:halt, :destroy], :stdout => true, :force => true do
      info "Executing 'halt/destroy' trigger"
      run "ansible-playbook -i #{inventory_file} --ask-sudo-pass #{vagrant_dir}/centos/playbooks/vagrant-halt_destroy.yml --extra-vars \"inventory_file=#{inventory_file}\""
    end
  end

  # Enable provisioning with CFEngine. CFEngine Community packages are
  # automatically installed. For example, configure the host as a
  # policy server and optionally a policy file to run:
  #
  # config.vm.provision "cfengine" do |cf|
  #   cf.am_policy_hub = true
  #   # cf.run_file = "motd.cf"
  # end
  #
  # You can also configure and bootstrap a client to an existing
  # policy server:
  #
  # config.vm.provision "cfengine" do |cf|
  #   cf.policy_server_address = "10.0.2.15"
  # end

  # Enable provisioning with Puppet stand alone.  Puppet manifests
  # are contained in a directory path relative to this Vagrantfile.
  # You will need to create the manifests directory and a manifest in
  # the file default.pp in the manifests_path directory.
  #
  # config.vm.provision "puppet" do |puppet|
  #   puppet.manifests_path = "manifests"
  #   puppet.manifest_file  = "site.pp"
  # end

  # Enable provisioning with chef solo, specifying a cookbooks path, roles
  # path, and data_bags path (all relative to this Vagrantfile), and adding
  # some recipes and/or roles.
  #
  # config.vm.provision "chef_solo" do |chef|
  #   chef.cookbooks_path = "../my-recipes/cookbooks"
  #   chef.roles_path = "../my-recipes/roles"
  #   chef.data_bags_path = "../my-recipes/data_bags"
  #   chef.add_recipe "mysql"
  #   chef.add_role "web"
  #
  #   # You may also specify custom JSON attributes:
  #   chef.json = { mysql_password: "foo" }
  # end

  # Enable provisioning with chef server, specifying the chef server URL,
  # and the path to the validation key (relative to this Vagrantfile).
  #
  # The Opscode Platform uses HTTPS. Substitute your organization for
  # ORGNAME in the URL and validation key.
  #
  # If you have your own Chef Server, use the appropriate URL, which may be
  # HTTP instead of HTTPS depending on your configuration. Also change the
  # validation key to validation.pem.
  #
  # config.vm.provision "chef_client" do |chef|
  #   chef.chef_server_url = "https://api.opscode.com/organizations/ORGNAME"
  #   chef.validation_key_path = "ORGNAME-validator.pem"
  # end
  #
  # If you're using the Opscode platform, your validator client is
  # ORGNAME-validator, replacing ORGNAME with your organization name.
  #
  # If you have your own Chef Server, the default validation client name is
  # chef-validator, unless you changed the configuration.
  #
  #   chef.validation_client_name = "ORGNAME-validator"
end
