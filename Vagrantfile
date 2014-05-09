# -*- mode: ruby -*-
# vi: set ft=ruby :

require 'yaml'
config_dir = File.dirname(File.expand_path(__FILE__))
config_dir = "#{config_dir}/.config"

data = YAML::load_file("#{config_dir}/config.yaml")
data = data['vagrant']
vm   = data['vm']

if !data['require_version'].nil? and data['require_version'].to_s != ''
    Vagrant.require_version "#{data['require_version']}"
end

Vagrant.configure("2") do |config|

    ## Virtual Machine
    if !vm['boot_timeout'].nil? and vm['boot_timeout'].to_i >= 0
        config.vm.boot_timeout = "#{vm['boot_timeout']}".to_i
    end

    config.vm.box = "#{vm['box']}"

    if !vm['box_url'].nil? and vm['box_url'].to_s != ''
        config.vm.box_url = "#{vm['box_url']}"
    end

    if !vm['box_check_update'].nil?
        config.vm.box_check_update = "#{vm['box_check_update']}"
    end

    if !vm['box_download_checksum'].nil? and vm['box_download_checksum'].to_s != ''
        config.vm.box_download_checksum = "#{vm['box_download_checksum']}"
    end

    if !vm['box_download_checksum_type'].nil? and vm['box_download_checksum_type'].to_s != ''
        config.vm.box_download_checksum_type = "#{vm['box_download_checksum_type']}"
    end

    if !vm['box_download_client_cert'].nil? and vm['box_download_client_cert'].to_s != ''
        config.vm.box_download_client_cert = "#{vm['box_download_client_cert']}"
    end

    if !vm['box_download_insecure'].nil?
        config.vm.box_download_insecure = "#{vm['box_download_insecure']}"
    end

    if !vm['box_version'].nil? and vm['box_version'].to_s != ''
        config.vm.box_version = "#{vm['box_version']}"
    end

    if !vm['graceful_halt_timeout'].nil? and vm['graceful_halt_timeout'].to_i >= 0
        config.vm.graceful_halt_timeout = "#{vm['graceful_halt_timeout']}".to_i
    end

    if !vm['guest'].nil? and vm['guest'].to_s != ''
        config.vm.guest = "#{vm['guest']}"
    end

    if !vm['hostname'].nil? and vm['hostname'].to_s != ''
        config.vm.hostname = "#{vm['hostname']}"
    end

    if !vm['post_up_message'].nil? and vm['post_up_message'].to_s != ''
        config.vm.post_up_message = "#{vm['post_up_message']}"
    end

    if !vm['usable_port_range'].nil? and !vm['usable_port_range'].empty?
        vm['usable_port_range'].each do |i, port|
            if port['from'].to_i > 0 and port['to'].to_i > 0
                config.vm.usable_port_range = (port['from']..port['to'])
            end
        end
    end

    ## Network
    if !vm['network'].nil? and !vm['network'].empty?
        
        ## Private
        if !vm['network']['private'].nil? and !vm['network']['private'].empty?
            vm['network']['private'].each do |i, val|
                ac = (!val['auto_config'].nil?) ? ac : true
                if !val['type'].nil? and val['type'] == 'dhcp'
                    config.vm.network "private_network",
                        type: "dhcp",
                        auto_config: ac
                end
                if !val['ip'].nil? and val['ip'].to_s != ''
                    config.vm.network "private_network",
                        ip: val['ip'].to_s,
                        auto_config: ac
                end
            end
        end

        ## Public
        if !vm['network']['public'].nil? and !vm['network']['public'].empty?
            vm['network']['public'].each do |i, val|
                ac = (!val['auto_config'].nil?) ? val['auto_config'] : true
                bd = (!val['bridge'].nil? and val['bridge'].to_s != '') ? val['bridge'].to_s : nil
                if !val['type'].nil? and val['type'] == 'dhcp'
                    config.vm.network "public_network",
                        type: "dhcp",
                        auto_config: ac,
                        :bridge => bd.to_s
                end
                if !val['ip'].nil? and val['ip'].to_s != ''
                    config.vm.network "public_network",
                        ip: val['ip'].to_s,
                        auto_config: ac,
                        :bridge => bd.to_s
                end
            end
        end

        ## Port forward
        if !vm['network']['forwarded_port'].nil? and !vm['network']['forwarded_port'].empty?
            vm['network']['forwarded_port'].each do |i, val|
                if val['guest'].to_i > 0 and val['host'].to_i > 0
                    _guest_ip = (!val['guest_ip'].nil? and val['guest_ip'].to_s != '') ? val['guest_ip'].to_s : nil
                    _host_ip = (!val['host_ip'].nil? and val['host_ip'].to_s != '') ? val['host_ip'].to_s : nil
                    _protocol  = (!val['protocol'].nil? and !val['protocol'].empty?) ? val['protocol'] : ['tcp']
                    _auto_correct  = (!val['auto_correct'].nil?) ? val['auto_correct'] : true
                    _protocol.each do |type|
                        config.vm.network "forwarded_port",
                            guest: val['guest'].to_i,
                            guest_ip: _guest_ip,
                            host: val['host'].to_i,
                            host_ip: _host_ip,
                            protocol: type.to_s,
                            auto_correct: _auto_correct
                    end
                end
            end
        end
    end

    ## Provider
    if !vm['provider'].nil? and !vm['provider'].empty?
        ## VirtualBox
        if !vm['provider']['virtualbox'].nil? and !vm['provider']['virtualbox'].empty?
            config.vm.provider :virtualbox do |virtualbox|
                vm['provider']['virtualbox'].each do |option, value|
                    value.each do |key, val|
                        virtualbox.customize ["#{option}", :id, "--#{key}", "#{val}"]
                    end
                end
            end
        end
    end

    ## Synced folders
    if !vm['synced_folder'].nil? and !vm['synced_folder'].empty?
        vm['synced_folder'].each do |i, folder|
            _create = (!folder['create'].nil?) ? folder['create'] : false
            _disabled = (!folder['disabled'].nil?) ? folder['disabled'] : false
            _group = (!folder['group'].nil? and folder['group'].to_s != '') ? folder['group'].to_s : nil
            _mount_options = (!folder['mount_options'].nil? and !folder['mount_options'].empty? != '') ? folder['mount_options'] : []
            _owner = (!folder['owner'].nil? and folder['owner'].to_s != '') ? folder['owner'].to_s : nil
            _type = (!folder['type'].nil? and folder['type'].to_s != '') ? folder['type'].to_s : nil

            if _type == 'nfs'
                _nfs_udp = (!folder['nfs_udp'].nil?) ? folder['nfs_udp'] : true
                _nfs_version = (!folder['nfs_version'].nil? and (folder['nfs_version'].to_s != '' or folder['nfs_version'].to_i > 0)) ? folder['nfs_version'] : true
                config.vm.synced_folder folder['source'].to_s, folder['target'],
                    create: _create,
                    disabled: _disabled,
                    mount_options: _mount_options,
                    type: _type,
                    nfs_udp: _nfs_udp,
                    nfs_version: _nfs_version
            elsif _type == 'rsync'
                _rsync__args = (!folder['rsync__args'].nil? and !folder['rsync__args'].empty?) ? folder['rsync__args'] : []
                _rsync__auto = (!folder['rsync__auto'].nil?) ? folder['rsync__auto'] : true
                _rsync__exclude = (!folder['rsync__exclude'].nil? and (folder['rsync__exclude'].to_s != '' or !folder['rsync__exclude'].empty?0)) ? folder['rsync__exclude'] : nil
                config.vm.synced_folder folder['source'].to_s, folder['target'],
                    create: _create,
                    disabled: _disabled,
                    group: _group,
                    mount_options: _mount_options,
                    owner: _owner,
                    type: _type,
                    rsync__args: _rsync__args,
                    rsync__auto: _rsync__auto,
                    rsync__exclude: _rsync__exclude
            elsif _type == 'smb'
                _smb_host = (!folder['smb_host'].nil? and folder['smb_host'].to_s != '') ? folder['smb_host'] : nil
                _smb_password = (!folder['smb_password'].nil? and folder['smb_password'].to_s != '') ? folder['smb_password'] : nil
                _smb_username = (!folder['smb_username'].nil? and folder['smb_username'].to_s != '') ? folder['smb_username'] : nil
                config.vm.synced_folder folder['source'].to_s, folder['target'],
                    create: _create,
                    disabled: _disabled,
                    group: _group,
                    mount_options: _mount_options,
                    owner: _owner,
                    type: _type,
                    smb_host: _smb_host,
                    smb_password: _smb_password,
                    smb_username: _smb_username
            else
                config.vm.synced_folder folder['source'].to_s, folder['target'],
                    create: _create,
                    disabled: _disabled,
                    group: _group,
                    mount_options: _mount_options,
                    owner: _owner,
                    type: _type
            end
        end
    end


    # puts config.vm.to_yaml
    # exit

end

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
# VAGRANTFILE_API_VERSION = "2"

# Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  # All Vagrant configuration is done here. The most common configuration
  # options are documented and commented below. For a complete reference,
  # please see the online documentation at vagrantup.com.

  # Every Vagrant virtual environment requires a box to build off of.
  # config.vm.box = "ubuntu/trusty32"

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
  # config.vm.network "private_network", ip: "192.168.33.10"

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
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  # config.vm.provider "virtualbox" do |vb|
  #   # Don't boot with headless mode
  #   vb.gui = true
  #
  #   # Use VBoxManage to customize the VM. For example to change memory:
  #   vb.customize ["modifyvm", :id, "--memory", "1024"]
  # end
  #
  # View the documentation for the provider you're using for more
  # information on available options.

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
  #   chef.json = { :mysql_password => "foo" }
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
# end
