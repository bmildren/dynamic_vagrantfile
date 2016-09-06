et ft=ruby :

require 'yaml'

unless Vagrant.has_plugin?("vagrant-hostmanager")
    raise "vagrant-hostmanager is not installed
         You can install this plugin using the following command: 'vagrant plugin install vagrant-hostmanager'
         For more information visit https://github.com/devopsgroup-io/vagrant-hostmanager"
end

Vagrant.configure(2) do |config|

mysql_master_group, mysql_slave_group, mysql_backup_group, mysql_manager_group, mysql_proxy_group = Array.new(5){ [] }
generic_group = Array.new

    servers = YAML.load_file(File.join(File.dirname(__FILE__), 'vagrant-servers.yml'))

    if !servers.empty?
        server_count = servers.length
        servers.each.with_index(1) do |managed_server, server_num|
            managed_server["role"].each do |role_config|
                case role_config
                when "mysql_master"
                    mysql_master_group.push(managed_server["name"])
                when "mysql_slave"
                    mysql_slave_group.push(managed_server["name"])
                when "mysql_backup"
                    mysql_backup_group.push(managed_server["name"])
                when "mysql_manager"
                    mysql_manager_group.push(managed_server["name"])
                when "mysql_proxy"
                    mysql_proxy_group.push(managed_server["name"])
                else
                    generic_group.push(managed_server["name"])
                end
            end

            config.hostmanager.enabled = true
            config.hostmanager.manage_host = false
            config.hostmanager.manage_guest = true
            config.hostmanager.ignore_private_ip = false
            config.hostmanager.include_offline = true

            config.vm.define managed_server["name"] do |srv|
                if managed_server.has_key?("box")
                    srv.vm.box = managed_server["box"]
                else
                    srv.vm.box = "ubuntu/trusty64"
                end

                if managed_server.has_key?("box_check_update")
                    srv.vm.box_check_update = managed_server["box_check_update"]
                end

                srv.vm.hostname = managed_server["name"]
                srv.vm.network :private_network, ip: "192.168.66.#{65+server_num}"
                srv.hostmanager.aliases = %w(#{managed_server["name"]}.localdomain)

                if managed_server.has_key?("provider")
                    config.vm.provider managed_server["provider"] do |v|
                        managed_server["provider_config"].each_pair do |key, value|
                            v.customize ["modifyvm", :id, "#{key}", "#{value}"]
                        end
                    end
                end

                if server_num == server_count
                    srv.vm.provision :ansible do |ansible|
                        ansible.groups = {
                            "mysql_master" => mysql_master_group,
                            "mysql_slave" => mysql_slave_group,
                            "mysql_backup" => mysql_backup_group,
                            "mysql_manager" => mysql_manager_group,
                            "mysql_proxy" => mysql_proxy_group,
                            "generic" => generic_group,
                            "mysql:children" => ["mysql_master", "mysql_slave", "mysql_backup"],
                            "db:children" => ["mysql", "mysql_manager", "mysql_proxy"]
                        }
                        ansible.playbook = "test.yml"
                        ansible.skip_tags = "nrpe"
                        ansible.limit = "all"
                    end
                end
            end
        end
    end
end
