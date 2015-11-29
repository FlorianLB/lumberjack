# -*- mode: ruby -*-
# vi: set ft=ruby :

require 'yaml'

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

def symbolize_keys(hash)
  hash.inject({}){|result, (key, value)|
    new_key = case key
              when String then key.to_sym
              else key
              end
    new_value = case value
                when Hash then symbolize_keys(value)
                else value
                end
    result[new_key] = new_value
    result
  }
end

settings = YAML::load_file ".lumberjack.yml"
settings = symbolize_keys(settings)

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = settings[:box_name]
  config.vm.box_url = settings[:box_url]

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.

  # IP Address of the VM
  config.vm.network :private_network, ip: settings[:ip]

  # Prevent SSH forwarded ports conflicts on host machine
  config.vm.network :forwarded_port, guest: 22, host: settings[:ssh_port], id: 'ssh'

  # If you want forward yourself a port, see below example
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  # Hostname of the VM (use in ansible inventory : ansible/vagrant_hosts)
  config.vm.hostname = settings[:host_name]

  # Deactivation of the default share
  config.vm.synced_folder ".", "/vagrant", id: "vagrant-root", disabled: true

  config.vm.synced_folder "./", "/var/www/app", :nfs => true

  config.vm.provider :virtualbox do |v|
    v.gui = false

    v.customize [ "modifyvm", :id, "--cpus", settings[:proc] ]
    v.customize [ "modifyvm", :id, "--memory", settings[:ram] ]
  end

  config.vm.provision "ansible" do |ansible|
    ansible.inventory_path = "ansible/inventories/vagrant"
    ansible.playbook = "ansible/site.yml"
    ansible.limit= "all"
    ansible.verbose = ""
  end


  # PLUGINS
  config.vm.provider :virtualbox do |v|
    config.vbguest.auto_update = true
    config.vbguest.no_remote = false
  end

end
