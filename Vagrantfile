require 'yaml'
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
    config.vm.box = "chef/centos-6.5"

    vagrant_loc = ENV['VAGRANT_LOC']
    vagrant_env = ENV['VAGRANT_ENV']
    nodes, managers, monitors = Array.new(3){ [] }

    if vagrant_env.nil?
        if vagrant_loc.nil?
            ansible_playbook = "config/ansible_playbooks/default.yml"
            server_config = "config/servers/servers.yml"
        else
            ansible_playbook = "#{vagrant_loc}/config/ansible_playbooks/default.yml"
            server_config = "#{vagrant_loc}/config/servers/servers.yml"
        end
    else
        if vagrant_loc.nil?
            ansible_playbook = "config/ansible_playbooks/#{vagrant_env}_playbook.yml"
            server_config = "config/servers/#{vagrant_env}_servers.yml"
        else
            ansible_playbook = "#{vagrant_loc}/config/ansible_playbooks/#{vagrant_env}_playbook.yml"
            server_config = "#{vagrant_loc}/config/servers/#{vagrant_env}_servers.yml"
        end
    end

    servers = YAML.load_file(server_config)
    last_server = servers.pop

    if !servers.empty? 
        servers.each do |new_server|
            new_server["role"].each do |role_config|
                case role_config
                when "node"
                    nodes.push(new_server["name"])
                when "manager"
                    managers.push(new_server["name"])
                when "monitor"
                    monitors.push(new_server["name"])
                end
            end

            config.vm.define new_server["name"] do |srv|
                srv.vm.hostname = new_server["name"]
                srv.vm.network :private_network, ip: new_server["ip"]
            end
        end
    end

    config.vm.define last_server["name"] do |srv|
        last_server["role"].each do |role_config|
           case role_config
           when "node"
               nodes.push(last_server["name"])
           when "manager"
               managers.push(last_server["name"])
           when "monitor"
               monitors.push(last_server["name"])
           end
        end

        srv.vm.hostname = last_server["name"]
        srv.vm.network :private_network, ip: last_server["ip"]
                
        srv.vm.provision :ansible do |ansible|
            ansible.groups = {
                "managers" => managers,
                "nodes" => nodes,
                "monitors" => monitors,
                "cluster_vms:children" => ["managers", "nodes"],
                "all_vms:children" => ["cluster_vms", "monitors"]
            }
            ansible.playbook = ansible_playbook
            ansible.limit = "all"
        end
    end   
end
