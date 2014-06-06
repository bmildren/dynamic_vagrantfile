require 'yaml'
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

    vagrant_loc = ENV['VAGRANT_LOC']
    vagrant_env = ENV['VAGRANT_ENV']
    db_masters, db_slaves, managers, monitors = Array.new(4){ [] }

    if vagrant_env.nil?
        if vagrant_loc.nil?
            ansible_playbook = "config/ansible/default_playbook.yml"
            server_config = "config/servers/default_servers.yml"
        else
            ansible_playbook = "#{vagrant_loc}/config/ansible/default_playbook.yml"
            server_config = "#{vagrant_loc}/config/servers/default_servers.yml"
        end
    else
        if vagrant_loc.nil?
            ansible_playbook = "config/ansible/#{vagrant_env}_playbook.yml"
            server_config = "config/servers/#{vagrant_env}_servers.yml"
        else
            ansible_playbook = "#{vagrant_loc}/config/ansible/#{vagrant_env}_playbook.yml"
            server_config = "#{vagrant_loc}/config/servers/#{vagrant_env}_servers.yml"
        end
    end

    servers = YAML.load_file(server_config)
    last_server = servers.pop

    if !servers.empty? 
        servers.each do |new_server|
            new_server["role"].each do |role_config|
                case role_config
                when "db_master"
                    db_masters.push(new_server["name"])
                when "db_slave"
                    db_slaves.push(new_server["name"])
                when "manager"
                    managers.push(new_server["name"])
                when "monitor"
                    monitors.push(new_server["name"])
                end
            end

            config.vm.define new_server["name"] do |srv|
                if new_server.has_key?("box")
                    srv.vm.box = new_server["box"]
                else
                    srv.vm.box = "chef/centos-6.5"
                end
                if new_server.has_key?("provider")
                    config.vm.provider new_server["provider"] do |v|
                        new_server["provider_config"].each_pair do |key, value|
                           v.customize ["modifyvm", :id, "#{key}", "#{value}"]
                        end
                    end
                end
                if new_server.has_key?("box_check_update")
                    srv.vm.box_check_update = new_server["box_check_update"]
                end
                srv.vm.hostname = new_server["name"]
                srv.vm.network :private_network, ip: new_server["ip"]
            end
        end
    end

    config.vm.define last_server["name"] do |srv|
        last_server["role"].each do |role_config|
           case role_config
           when "db_master"
               db_masters.push(last_server["name"])
           when "db_slave"
               db_slaves.push(last_server["name"])
           when "manager"
               managers.push(last_server["name"])
           when "monitor"
               monitors.push(last_server["name"])
           end
        end

        if last_server.has_key?("box")
            srv.vm.box = last_server["box"]
        else
            srv.vm.box = "chef/centos-6.5"
        end
        if last_server.has_key?("provider")
            config.vm.provider last_server["provider"] do |v|
                last_server["provider_config"].each_pair do |key, value|
                    v.customize ["modifyvm", :id, "#{key}", "#{value}"]
                end
            end
        end
        if last_server.has_key?("box_check_update")
            srv.vm.box_check_update = last_server["box_check_update"]
        end
        srv.vm.hostname = last_server["name"]
        srv.vm.network :private_network, ip: last_server["ip"]
                
        srv.vm.provision :ansible do |ansible|
            ansible.groups = {
                "db_masters" => db_masters,
                "db_slaves" => db_slaves,
                "managers" => managers,
                "monitors" => monitors,
                "db_nodes:children" => ["db_masters", "db_slaves"],
                "db_cluster:children" => ["managers", "db_nodes"],
                "mng_nodes:children" => ["managers", "monitors"]
            }
            ansible.playbook = ansible_playbook
            ansible.limit = "all"
        end
    end   
end
