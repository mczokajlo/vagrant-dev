# -*- mode: ruby -*-
# vi: set ft=ruby :

require 'yaml'
config_dir = File.dirname(File.expand_path(__FILE__))
config_dir = "#{config_dir}/.config"

data = YAML::load_file("#{config_dir}/vagrant.yaml")
data = data['vagrant']
vm   = data['vm']

if !data['require_version'].nil? and data['require_version'].to_s != ''
    Vagrant.require_version "#{data['require_version'].to_s}"
end

if !data['vm']['provider']['virtualbox'].empty?
    ENV['VAGRANT_DEFAULT_PROVIDER'] = 'virtualbox'
end

Vagrant.configure("2") do |config|

    ## Virtual Machine
    if !vm['boot_timeout'].nil? and vm['boot_timeout'].to_i >= 0
        config.vm.boot_timeout = vm['boot_timeout'].to_i
    end

    config.vm.box = "#{vm['box']}"

    if !vm['box_url'].nil? and vm['box_url'].to_s != ''
        config.vm.box_url = vm['box_url'].to_s
    end

    if !vm['box_check_update'].nil?
        config.vm.box_check_update = vm['box_check_update']
    end

    if !vm['box_download_checksum'].nil? and vm['box_download_checksum'].to_s != ''
        config.vm.box_download_checksum = vm['box_download_checksum'].to_s
    end

    if !vm['box_download_checksum_type'].nil? and vm['box_download_checksum_type'].to_s != ''
        config.vm.box_download_checksum_type = vm['box_download_checksum_type'].to_s
    end

    if !vm['box_download_client_cert'].nil? and vm['box_download_client_cert'].to_s != ''
        config.vm.box_download_client_cert = vm['box_download_client_cert'].to_s
    end

    if !vm['box_download_insecure'].nil?
        config.vm.box_download_insecure = vm['box_download_insecure']
    end

    if !vm['box_version'].nil? and vm['box_version'].to_s != ''
        config.vm.box_version = vm['box_version'].to_s
    end

    if !vm['graceful_halt_timeout'].nil? and vm['graceful_halt_timeout'].to_i >= 0
        config.vm.graceful_halt_timeout = vm['graceful_halt_timeout'].to_i
    end

    if !vm['guest'].nil? and vm['guest'].to_s != ''
        config.vm.guest = vm['guest'].to_s
    end

    if !vm['hostname'].nil? and vm['hostname'].to_s != ''
        config.vm.hostname = vm['hostname'].to_s
    end

    if !vm['post_up_message'].nil? and vm['post_up_message'].to_s != ''
        config.vm.post_up_message = vm['post_up_message'].to_s
    end

    if !vm['usable_port_range'].nil? and !vm['usable_port_range'].empty?
        vm['usable_port_range'].each do |i, port|
            if port['from'].to_i > 0 and port['to'].to_i > 0
                config.vm.usable_port_range = (port['from'].to_i..port['to'].to_i)
            end
        end
    end

    ## Network
    if !vm['network'].nil? and !vm['network'].empty?
        
        ## Private
        if !vm['network']['private'].nil? and !vm['network']['private'].empty?
            vm['network']['private'].each do |i, val|
                _auto_config = (!val['auto_config'].nil?) ? val['auto_config'] : true
                if !val['type'].nil? and val['type'] == 'dhcp'
                    config.vm.network 'private_network',
                        type: 'dhcp',
                        auto_config: _auto_config
                end
                if !val['ip'].nil? and val['ip'].to_s != ''
                    config.vm.network 'private_network',
                        ip: val['ip'].to_s,
                        auto_config: _auto_config
                end
            end
        end

        ## Public
        if !vm['network']['public'].nil? and !vm['network']['public'].empty?
            vm['network']['public'].each do |i, val|
                _auto_config = (!val['auto_config'].nil?) ? val['auto_config'] : true
                _bridge = (!val['bridge'].nil? and val['bridge'].to_s != '') ? val['bridge'].to_s : nil
                if !val['type'].nil? and val['type'] == 'dhcp'
                    config.vm.network 'public_network',
                        type: 'dhcp',
                        auto_config: _auto_config,
                        :bridge => _bridge
                end
                if !val['ip'].nil? and val['ip'].to_s != ''
                    config.vm.network 'public_network',
                        ip: val['ip'].to_s,
                        auto_config: _auto_config,
                        :bridge => _bridge
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
                        config.vm.network 'forwarded_port',
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
            config.vm.provider 'virtualbox' do |virtualbox|
                vm['provider']['virtualbox'].each do |option, value|
                    value.each do |key, val|
                        virtualbox.customize [option, :id, "--#{key}", val]
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
                _smb_host = (!folder['smb_host'].nil? and folder['smb_host'].to_s != '') ? folder['smb_host'].to_s : nil
                _smb_password = (!folder['smb_password'].nil? and folder['smb_password'].to_s != '') ? folder['smb_password'].to_s : nil
                _smb_username = (!folder['smb_username'].nil? and folder['smb_username'].to_s != '') ? folder['smb_username'].to_s : nil
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

    ## SSH
    if !data['ssh'].nil? and !data['ssh'].empty?
        ssh = data['ssh']

        if !ssh['username'].nil? and ssh['username'].to_s != ''
            config.ssh.username = ssh['username'].to_s
        end

        if !ssh['password'].nil? and ssh['password'].to_s != ''
            config.ssh.password = ssh['password'].to_s
        end

        if !ssh['host'].nil? and ssh['host'].to_s != ''
            config.ssh.host = ssh['host'].to_s
        end

        if !ssh['port'].nil? and ssh['port'].to_i > 0
            config.ssh.port = ssh['host'].to_i
        end

        if !ssh['guest_port'].nil? and ssh['guest_port'].to_i > 0
            config.ssh.guest_port = ssh['guest_port'].to_i
        end

        if !ssh['private_key_path'].nil? and ssh['private_key_path'].to_s != ''
            config.ssh.private_key_path = ssh['private_key_path'].to_s
        end

        if !ssh['forward_agent'].nil?
            config.ssh.forward_agent = ssh['forward_agent']
        end

        if !ssh['forward_x11'].nil?
            config.ssh.forward_x11 = ssh['forward_x11']
        end

        if !ssh['insert_key'].nil?
            config.ssh.insert_key = ssh['insert_key']
        end

        if !ssh['proxy_command'].nil? and ssh['proxy_command'].to_s != ''
            config.ssh.proxy_command = ssh['proxy_command'].to_s
        end

        if !ssh['shell'].nil? and ssh['shell'].to_s != ''
            config.ssh.shell = ssh['shell'].to_s
        end

    end

    ## Vagrant
    if !data['vagrant'].nil? and !data['vagrant'].empty?
        vagran = data['vagrant']

        if !vagran['host'].nil? and vagran['host'].to_s != ''
            config.vagrant.host = vagran['host'].gsub(":", "").intern
        end

    end

end
