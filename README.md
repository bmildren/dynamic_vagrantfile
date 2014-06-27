vagrant_wrapper
===============

Wrapper around vagrant and ansible.

By default it's configured to use virtual box, ready-made boxes can be referenced in the config and automatically pulled using vagrant from here: https://vagrantcloud.com/discover/featured.

To use run:
vagrant_wrapper <command> <environment> optional: <machine>

e.g. "vagrant_wrapper start" to run the default config or "vagrant_wrapper start myenv"

The environment is user definable but vagrant_wrapper will expect two configuration files named in line with the environment, one in the config/ansible/ dir (e.g. myenv_playbook.yml) and one in the config/servers/ dir (e.g. myenv_servers.yml)

Available commands:

start (vagrant up),
connect (vagrant ssh),
status (vagrant status),
refresh (vagrant provision),
reload (vagrant reload),
stop (vagrant halt),
destroy (vagrant destroy --force)
