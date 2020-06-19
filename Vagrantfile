# -*- mode: ruby -*-
# # vi: set ft=ruby :

require 'yaml'
dir = File.dirname(File.expand_path(__FILE__))
conf_file = "#{dir}/group_vars/config-vagrant.yaml"

if !File.exist?("#{conf_file}")
  raise 'Provisioning file yaml is missing! ARRGHHH! Missing execution parameters! ARRGHH!!! Stopping provision... Damn it!'
end
vagrantconf = YAML::load_file("#{conf_file}")

BRIDGE_NET = vagrantconf['vagrant_ip']
DOMAIN = vagrantconf['vagrant_domain_name']
MEMORY = vagrantconf['vagrant_memory']

vmnodes=[
  {
    :hostname => "gitlab-server." + "#{DOMAIN}",
    :ip => "#{BRIDGE_NET}" + "15",
    :ram => 4096,
    :sport => 443,
    :dport => 8443
  },
  {
    :hostname => "gitlab-runner." + "#{DOMAIN}",
    :ip => "#{BRIDGE_NET}" + "16",
    :ram => "#{MEMORY}",
    :sport => 22,
    :dport => 2220
   }
]

Vagrant.configure("2") do |config|
    config.vm.synced_folder ".", vagrantconf['vagrant_directory'], :mount_options => ["dmode=777", "fmode=666"]
        vmnodes.each do |createvm|
            config.vm.define createvm[:hostname] do |vmparam|
		vmparam.vm.box = vagrantconf['vagrant_box']
		vmparam.vm.hostname = createvm[:hostname]
                vmparam.vm.network "private_network", ip: createvm[:ip]
                vmparam.vm.network "forwarded_port", guest: createvm[:sport], host: createvm[:dport]
                vmparam.vm.provider "virtualbox" do |virtualbox|
                virtualbox.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
		virtualbox.cpus = vagrantconf['vagrant_cpu']
		virtualbox.memory = createvm[:ram]
                virtualbox.name = createvm[:hostname]
                config.ssh.insert_key = false
            end # virtualbox end
        end # vmparam end
    end # createvm end
end # config end
