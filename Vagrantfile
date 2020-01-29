Vagrant.require_version ">= 1.7.0"

$script_lxdserver = <<-SCRIPT
test -e /usr/bin/pip3 || (apt -y update && apt install -y python3-pip)
#sudo chmod 0600 /vagrant/sshkey/id_rsa_dev
SCRIPT

$script_controller = <<-SCRIPT
sudo chmod o-w /vagrant/ansible
sudo chmod o-w /vagrant
SCRIPT


def import_config
    require 'yaml'
    current_dir    = File.dirname(File.expand_path(__FILE__))

    begin
        YAML.load_file("#{current_dir}/vagrant_config.local.yml")
    rescue
        YAML.load_file("#{current_dir}/vagrant_config.yml")
    end
end

dev_config = import_config

Vagrant.configure("2") do |config|
    if dev_config['provider'] == 'libvirt'
        config.vm.provider "libvirt"
    else
        config.vm.provider "virtualbox"
    end

    config.vm.box = "generic/ubuntu1910"
    


    ansi_provision = lambda do |playbook, provider|
        config.vm.provision "ansible_local" do |ansible|
              ansible.playbook       = "#{playbook}"
              ansible.config_file = "ansible/ansible.cfg"
              ansible.verbose        = true
              ansible.limit          = "all"
              ansible.inventory_path = "inventories/vagrant_#{provider}"
              ansible.verbose        = ""
        end
    end

    config.vm.define "lxdserver" do |machine|
        machine.vm.hostname = "lxdserver"
        machine.vm.network "private_network", ip: "172.16.200.10"
        if dev_config['provider'] == 'libvirt'

            machine.vm.provider :libvirt do |lv|
                lv.machine_virtual_size = dev_config['lxdserver_disk']
                lv.memory = dev_config['lxdserver_ram']
                lv.cpus = dev_config['lxdserver_cpu']

                machine.vm.synced_folder './', '/vagrant', type: 'nfs'
                ansi_provision.call("site_dev.yml", "libvirt")

            end
        elsif dev_config['provider'] == 'virtualbox'
            unless Vagrant.has_plugin?("vagrant-disksize")
                raise 'vagrant-disksize is not installed!'
            end
            machine.vm.provider :virtualbox do |vb|
                vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]

                config.disksize.size = '%dGB' % [dev_config['lxdserver_disk']]
                vb.memory = dev_config['lxdserver_ram']
                vb.cpus = dev_config['lxdserver_cpu']
                vb.customize ["modifyvm", :id, "--audio", "none"]
                machine.vm.synced_folder ".", "/vagrant"
                ansi_provision.call("site_dev.yml", "virtualbox")
            end
        end

        machine.vm.provision "shell", inline: $script_lxdserver
    end


end

