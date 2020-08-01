# Ansible script patterns

[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.3522974.svg)](https://doi.org/10.5281/zenodo.3522974)

<!--TOC-->

# Setup

## Ansible - Installation on control machine

* on mac or linux, do `pip install ansible`.
* For any OS with anaconda Python distribution, I make a separate conda environment named `ansible`, since it requires pip to install, so you keep it separate from other packages installed with conda (mixing conda and pip generally means trouble over time).

    * if on catalina, might need to open `bash` from `zsh` shell, and if so, will likely also need to move your anaconda init from `~/.bash_profile` to `~/.bashrc`.
    * `conda create --name ansible python=3`
    * `conda activate ansible`
    * `pip install ansible`

* for other ways to install: [http://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html](http://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)

## Ansible - Installation on a target

* [http://docs.ansible.com/ansible/latest/user_guide/intro_getting_started.html](http://docs.ansible.com/ansible/latest/user_guide/intro_getting_started.html)
* put your control machine’s SSH public key in `~/.ssh/authorized_keys` on the target machine in the home folder of the user you'll use to run ansible commands on the target. (permissions: `~/.ssh` folder should be 700; `authorized_keys` should be 600).
* make sure that python 2 or 3 are installed.  From installation guide:

    * By default, Ansible uses the python interpreter located at /usr/bin/python to run its modules. However, some Linux distributions may only have a Python 3 interpreter installed to /usr/bin/python3 by default. On those systems, you may see an error like:

        * `"module_stdout": "/bin/sh: /usr/bin/python: No such file or directory\r\n"`

    * you can either set the `ansible_python_interpreter` inventory variable (see Working with Inventory) to point at your interpreter or you can install a Python 2 interpreter for modules to use. You will still need to set `ansible_python_interpreter` if the Python 2 interpreter is not installed to `/usr/bin/python`.

- if it is a new installation, also update `/etc/hosts` so that it includes your host name in the entry for “127.0.0.1” (example of having added hostname “research”):

    - `127.0.0.1 localhost.localdomain localhost research`
    - NOTE: if your hostname (run `hostname` to see what it is) isn’t a for-real DNS name, if you don’t update `/etc/hosts` to include your server name (example: “research”) in your set of DNS names for 127.0.0.1, sudo will be SLOW: https://ubuntuforums.org/showthread.php?t=1155261

- And, a random note: If you are installing on a local VM where you'll assign an IP address (I ran into it in VMWare Fusion), but have no DNS name, but you want to make a DNS name by adding it to `/etc/hosts` on your host computer, don't put underscores in the host name you make up.  Far as I can tell, Apache 2.4 will not accept requests to the hostname with underscores in it, but will accept requests to the IP address of the VM, and will accept requests to hostnames with no underscores.  There might be a way to get this to work, but I do not know it. So, for example:

    - GOOD: `ubuntu.local`
    - BAD: `ubuntu_server_lts.local`

# `research` patterns

Research server playbook is `research.yml`.

To fire up a sourcenet/context server, do the `research.yml` playbook, then the `only_context_dev.yml` playbook.

## `research` quick start

The following steps will install all services on the server whose DNS name is `research.local`:

- go into `ansible-patterns/host_vars` and rename `research.local.yml` to `research.local`.  Inside, make sure to set:

    - `ansible_user`
    - `ansible_become_password`
    - `server_ip_address`
    - `server_webroot_folder` (defaults to ubuntu default of `/var/www/html`)
    - `jupyterhub_configproxy_auth_token` (32 character random hexadecimal number: `openssl rand -hex 32`)

- set `research.local` to point to the IP address of the server you want to install to in your hosts file (`/etc/hosts` on unix-like machines).
- run the "research.yml" playbook: in `ansible-patterns` root, run `ansible-playbook research.yml -i ./hosts.yml`
- To also install the `context` set of applications, run the `research.yml` playbook, then the `only_context_dev.yml` playbook: `ansible-playbook only_context_dev.yml -i ./hosts.yml`

This quick start assumes that you are making a server whose DNS name is "research.local".  If you want a different DNS name, for example "ubuntu.local":

- when you copy `research.local.yml`, name the copy the DNS name you want (`ubuntu.local`).
- Inside your copy of `research.local.yml` (named `ubuntu.local` for this example):

    - set `server_host_name` to your server's internal host name - "ubuntu"?
    - set `server_name` to your server's DNS name for external connections - "ubuntu.local" in this example.

- Update the file `hosts.yml` to substitute your preferred DNS name ("ubuntu.local" in this example) for all mentions of "research.local".

## `research` notes

- must update:

    - hosts.yml

        - need to update to match the particular layout of servers you intend to use.
        - default is all installed on the same server, named research.local.
        - Note: more complex configurations will require manual configuration beyond what these scripts provide.

    - /host_vars

        - each host needs a file named its DNS name to hold variables.
        - use the `research.local.yml` file as a template for variables related to each of your hosts.
        - This includes:

            - ansible_user
            - ansible_become_password
            - server_ip_address
            - jupyterhub_configproxy_auth_token (32 character random hexadecimal number: `openssl rand -hex 32`)

        - Notes on specific variables:

            - django_superuser_password: The command that sets the django superuser password uses a shell script, and so the password value here can't contain:
                - "$"
                - open square bracket without closed square bracket ("[").

    - vault password(s)

        - if you want to encrypt either files or variable values within yml files, you'll need to have one or more vault passwords to use as the symmetric encryption key.
        - if you need your deployment to run unattended, you'll want to create one or more files for your vault passwords, then direct the ansible-playbook command to them when you deploy.

            - these can be located wherever you want.

    - ansible config:

        - one way to store a single vault password is to store its location in ansible config, in  `vault_password_file`.
        - default locations:

            - `/etc/ansible/ansible.cfg`
            - `~/.ansible.cfg`

    - if installing rstudio, `roles/rstudio/tasks/rstudio_variables.yml`

        - add rstudio version to download (will have latest as of last changes to that file, might be outdated).

# Ansible Notes

## Getting started

* Ansible documentation: [http://docs.ansible.com/ansible/latest/index.html](http://docs.ansible.com/ansible/latest/index.html)

    * [https://www.ansible.com/resources/get-started](https://www.ansible.com/resources/get-started)
    * [http://docs.ansible.com/ansible/latest/user_guide/playbooks_intro.html](http://docs.ansible.com/ansible/latest/user_guide/playbooks_intro.html)
    * [http://docs.ansible.com/ansible/latest/user_guide/modules.html](http://docs.ansible.com/ansible/latest/user_guide/modules.html)
    * [http://docs.ansible.com/ansible/latest/user_guide/intro_patterns.html](http://docs.ansible.com/ansible/latest/user_guide/intro_patterns.html)

* Playbook examples:

    * [https://github.com/ansible/ansible-examples](https://github.com/ansible/ansible-examples)
    * [https://dotlayer.com/how-to-use-an-ansible-playbook-to-install-wordpress/](https://dotlayer.com/how-to-use-an-ansible-playbook-to-install-wordpress/)

## inventory file

* Inventory file is where ansible looks to see which servers it is controlling.
* default location is `/etc/ansible/hosts`

    * on work computer, mine is in `/Users/jonathanmorgan/Documents/work/ansible/research`

* when you run ansible, you can specify a different file with the `-i <inventory_file_path>` command.
* ansible inventory and playbook files are generally implemented in YAML.  Example simple inventory:

        all:
          vars:
            ansible_user: jonscomo
            ansible_python_interpreter: /usr/bin/python3
          hosts:
            172.16.111.133
        all:
          hosts:
            172.16.111.133:
              ansible_user: jonscomo
              ansible_python_interpreter: /usr/bin/python3

* any grouping from “all” down can have a set of “vars” associated with it.
* if a variable is defined in multiple places, you can also 
* specific variables:

    * ansible_user: user ansible uses when connecting.
    * ansible_python_interpreter: path to Python to use (either in general, if in all—>vars, or per host if under host name).
    * you can also specify variable values for a given host in a separate file:

        * host variable files are located in ./host_vars, relative to the location of your playbook.
        * each host’s variables are stored in a file named just their host name inside host_vars.
        * variables are stored in the root of a YAML file.  Simple example:

                # default variables for host research.local
                ansible_user: jonathanmorgan
                ansible_python_interpreter: /usr/bin/python3
                ansible_become: yes
                ansible_become_user: root
                ansible_become_password: today123

* details: [http://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html](http://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html)

## ansible command basics

* once you have inventory file, the most basic tests are to ping, then run a simple command:

        ansible all -m ping -i ./hosts.yaml
        ansible all -a "/bin/echo hello" -i ./hosts.yaml

* options:

    * `-i <inventory_file_path>`
    * `-u <username>` (on remote machine)

* sudo examples:

        # as bruce
        $ ansible all -m ping -u bruce
        # as bruce, sudoing to root
        $ ansible all -m ping -u bruce --sudo
        # as bruce, sudoing to batman
        $ ansible all -m ping -u bruce --sudo --sudo-user batman
         
        # With latest version of ansible `sudo` is deprecated so use become
        # as bruce, sudoing to root
        $ ansible all -m ping -u bruce -b
        # as bruce, sudoing to batman
        $ ansible all -m ping -u bruce -b --become-user batman
    
    * From: [http://docs.ansible.com/ansible/latest/user_guide/intro_getting_started.html](http://docs.ansible.com/ansible/latest/user_guide/intro_getting_started.html)

## overview

* hosts, and then you have a single playbook, with different sets of tasks broken out into roles.
* good simple example (lamp stack): [https://github.com/ansible/ansible-examples/tree/master/lamp_simple](https://github.com/ansible/ansible-examples/tree/master/lamp_simple)

## configuration

* Overview: [https://docs.ansible.com/ansible/latest/installation_guide/intro_configuration.html](https://docs.ansible.com/ansible/latest/installation_guide/intro_configuration.html)
* Config parameter documentation: [https://docs.ansible.com/ansible/latest/reference_appendices/config.html](https://docs.ansible.com/ansible/latest/reference_appendices/config.html)

    * Notable examples:

        * vault_password_file - path to vault password file ([https://docs.ansible.com/ansible/latest/reference_appendices/config.html#default-vault-password-file](https://docs.ansible.com/ansible/latest/reference_appendices/config.html#default-vault-password-file))

* Example INI file: [https://raw.githubusercontent.com/ansible/ansible/devel/examples/ansible.cfg](https://raw.githubusercontent.com/ansible/ansible/devel/examples/ansible.cfg)
* Command to view config: [http://docs.ansible.com/ansible/latest/cli/ansible-config.html](http://docs.ansible.com/ansible/latest/cli/ansible-config.html)
* Default INI-format ansible.cfg file locations:

        /etc/ansible/ansible.cfg
        ~/.ansible.cfg

## playbooks

* documentation:

    * doc overview: [http://docs.ansible.com/ansible/latest/user_guide/playbooks.html](http://docs.ansible.com/ansible/latest/user_guide/playbooks.html)
    * playbook intro: [http://docs.ansible.com/ansible/latest/user_guide/playbooks_intro.html](http://docs.ansible.com/ansible/latest/user_guide/playbooks_intro.html)

* running a playbook:

    * `ansible-playbook <playbook-file-name>`
    * with inventory file: `ansible-playbook <playbook-file-name> -i ./hosts.yml`
    * and with vault id: `ansible-playbook research.yml -i ./hosts.yml --vault-id dev@~/.ansible_vault`

* variables:

    * [https://docs.ansible.com/ansible/latest/user_guide/playbooks_variables.html#playbooks-variables](https://docs.ansible.com/ansible/latest/user_guide/playbooks_variables.html#playbooks-variables)

* privilege escalation: [http://docs.ansible.com/ansible/latest/user_guide/become.html](http://docs.ansible.com/ansible/latest/user_guide/become.html)

    * example in a host’s variables to tell it, by default, to elevate (by default using sudo), to root, using password:

            # default variables for host research.local
            ansible_become: yes
            ansible_become_user: root
            ansible_become_password: today123

* Playbooks for anaconda python (coming soon to this repo!)

    * https://github.com/andrewrothstein
    * https://github.com/andrewrothstein/ansible-anaconda

## Roles

* Documentation: [https://docs.ansible.com/ansible/latest/user_guide/playbooks_reuse_roles.html#roles](https://docs.ansible.com/ansible/latest/user_guide/playbooks_reuse_roles.html#roles)
* Roles are sets of tasks that can be combined in different ways to create various configurations on a particular server or set of servers.
* Roles by default are stored in a folder named “roles” in the directory where your playbook lives.  Each role’s folder includes:

    * /handlers - handler definitions (reusable tasks that can be invoked from tasks lists) for the role.  Main file is “main.yml”, then it can include others if you want to separate handlers out for organization.
    * /tasks - list of tasks to implement for a given role.  The main file is “main.yml”, then it can include others if you want to separate handlers out for organization.
    * /templates - files that are templates for configuration or other uses for the role.
    * /vars - files to hold variables for a role.  Main file is “main.yml”.

* A playbook decides which roles are invoked based on role tasks.  A given role task will contain a name (of course), a “hosts” field that either includes a host, a list of hosts, or a hosts group from your inventory file, and then the role to run against all matching hosts.

    * Example inventory file:

            all:
              hosts:
                research.local
              children:
                new_servers:
                  hosts:
                    research.local
    * Example role task to run the “new_server” role against servers in “new_servers” group:

            - name: set up new servers
              hosts: new_servers
              roles:
                - new_server

        * This combination will cause the tasks in `./roles/new_server/tasks/main.yml` to be run against the hosts in group “new_servers”, so in this example, just the host “research.local”.

## Vault

* Allows you to encrypt either variable values or files using a passphrase.
* General vault documentation: [http://docs.ansible.com/ansible/latest/user_guide/vault.html](http://docs.ansible.com/ansible/latest/user_guide/vault.html)
* ansible-vault command ([https://docs.ansible.com/ansible/latest/cli/ansible-vault.html](https://docs.ansible.com/ansible/latest/cli/ansible-vault.html)) lets you encrypt files or variable values within a value based on a secret encryption password.
* to start, you need a vault password.  This can be provided on the command line, or stored in a file.

    * create a strong password.
    * store the password and only the password in the first and only line of a file in your user's home folder named "`.ansible_vault`" (so "`~/.ansible_vault`").
    * all share same password:

        * command line (all share same password):

            * `ansible-playbook site.yml --vault-id ~/.ansible_vault`

        * in config INI-format file (either `/etc/ansible/ansible.cfg` or `~/.ansible.cfg` by default, or provided by the -c option),  create an ansible configuration file that contains the following:

                [defaults]
                vault_password_file = <absolute_path_to_vault_password_file>:

    * different password per ID (command line only):

        * `ansible-playbook site.yml --vault-id dev@~/.ansible_vault --vault-id prod@~/.ansible_prod_vault`

* basics:

    * [https://docs.ansible.com/ansible/latest/user_guide/playbooks_vault.html](https://docs.ansible.com/ansible/latest/user_guide/playbooks_vault.html)
    * encrypt a value to include in a YAML file variable:
        * [http://docs.ansible.com/ansible/latest/cli/ansible-vault.html#ansible-vault-encrypt-string](http://docs.ansible.com/ansible/latest/cli/ansible-vault.html#ansible-vault-encrypt-string)
        * `ansible-vault encrypt_string --vault-password-file <path_to_password_file> --encrypt-vault-id default '<string_to_encrypt>' --name '<variable_name>'`
        * Example: `ansible-vault encrypt_string --vault-password-file ~/.ansible_vault --encrypt-vault-id default '<password>' --name ansible_become_password`
        * Example output:

                ansible_become_password: !vault |
                $ANSIBLE_VAULT;1.1;AES256
                35633331303937613165656461336336343638643562383435383162303662663634366337353730
                3535666630666130656238313630363662383766303131350a303031613934646339313836393932
                30353838646265663535653036353339633664383033623338316236353439613930396335653634
                6332653732643937330a336532363265313731636538646438396162656666366439303561316465
                6265 
                Encryption successful

* multiple vault passwords:
    * you can assign IDs when you encrypt, then specify where to get passwords for each ID in your playbook command.
        * [https://docs.ansible.com/ansible/2.4/vault.html#providing-vault-passwords](https://docs.ansible.com/ansible/2.4/vault.html#providing-vault-passwords)
        * id “dev”, from a file “`~/.ansible_vault`": `ansible-playbook research.yaml -i ./hosts.yaml --vault-id dev@~/.ansible_vault`
    * encrypt a value to include in a YAML file variable, using a particular ID:
        * [http://docs.ansible.com/ansible/latest/cli/ansible-vault.html#ansible-vault-encrypt-string](http://docs.ansible.com/ansible/latest/cli/ansible-vault.html#ansible-vault-encrypt-string)
        * `ansible-vault encrypt_string '<string_to_encrypt>' --name '<variable_name>' --vault-password-file <id_name>@<where_to_get_password> --encrypt-vault-id <id_name>`
        * example: `ansible-vault encrypt_string 'today123' --name 'the_dev_secret' --vault-password-file dev@~/.ansible_vault --encrypt-vault-id dev`

## Variables

* Documentation: [http://docs.ansible.com/ansible/latest/user_guide/playbooks_variables.html](http://docs.ansible.com/ansible/latest/user_guide/playbooks_variables.html)
* Examples:

    * [http://www.dasblinkenlichten.com/ansible-roles-and-variables/](http://www.dasblinkenlichten.com/ansible-roles-and-variables/)

## Tasks:

* notify:

    * use notify to tell ansible that on success for a given task, it should notify the specified handler to run.
    * Handlers can be notified many times within a role or playbook, but will only be run once at the end.
    * Example:

        * Handler:

                - name: restart postgresql
                  service:
                    name: postsgresql
                    state: restarted
        * Notify in task:

                # allow database username/password connections.
                - name: Set listen_addresses for postgresql
                  lineinfile:
                    path: '/etc/postgresql/{{server_pgsql_version}}/main/pg_hba.conf'
                    line: 'host all all  0.0.0.0/0 md5'
                  notify: restart postgresql
* conditionals:

    * Documentation:

        * [https://docs.ansible.com/ansible/latest/user_guide/playbooks_conditionals.html](https://docs.ansible.com/ansible/latest/user_guide/playbooks_conditionals.html)
        * comparison operators: [http://jinja.pocoo.org/docs/dev/templates/#comparisons](http://jinja.pocoo.org/docs/dev/templates/#comparisons)

    * Example:
    * 

* module index: [https://docs.ansible.com/ansible/latest/modules/modules_by_category.html](https://docs.ansible.com/ansible/latest/modules/modules_by_category.html)
* network

    * [http://docs.ansible.com/ansible/latest/modules/list_of_net_tools_modules.html](http://docs.ansible.com/ansible/latest/modules/list_of_net_tools_modules.html)
    * get_url - [http://docs.ansible.com/ansible/latest/modules/get_url_module.html](http://docs.ansible.com/ansible/latest/modules/get_url_module.html)
    * uri - [http://docs.ansible.com/ansible/latest/modules/uri_module.html](http://docs.ansible.com/ansible/latest/modules/uri_module.html)

* managing SSH: [https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/](https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/)
* deploying keys from encrypted files in ansible:

    * [https://opensource.com/article/16/12/devops-security-ansible-vault](https://opensource.com/article/16/12/devops-security-ansible-vault)

* SSL certs/keys:

    * openssl_* modules (see below, also):

        *  Documentation: [https://docs.ansible.com/ansible/latest/modules/list_of_crypto_modules.html](https://docs.ansible.com/ansible/latest/modules/list_of_crypto_modules.html) 
        * openssl_* modules for creating keys: [https://www.jeffgeerling.com/blog/2017/generating-self-signed-openssl-certs-ansible-24s-crypto-modules](https://www.jeffgeerling.com/blog/2017/generating-self-signed-openssl-certs-ansible-24s-crypto-modules)

    * created using shell and openssl command:

        * [https://www.jeffgeerling.com/blog/2017/self-signed-certificates-ansible-local-testing-nginx](https://www.jeffgeerling.com/blog/2017/self-signed-certificates-ansible-local-testing-nginx)
        * [https://serialized.net/2013/04/simply-generating-self-signed-ssl-certs-with-ansible/](https://serialized.net/2013/04/simply-generating-self-signed-ssl-certs-with-ansible/)

* generate /etc/hosts with ansible: [https://gist.github.com/rothgar/8793800](https://gist.github.com/rothgar/8793800)
* security: [https://ryaneschinger.com/blog/securing-a-server-with-ansible/](https://ryaneschinger.com/blog/securing-a-server-with-ansible/)
* database:

    * postgresql: [https://docs.ansible.com/ansible/latest/modules/list_of_database_modules.html#postgresql](https://docs.ansible.com/ansible/latest/modules/list_of_database_modules.html#postgresql)

* text file lines:

    * [https://stackoverflow.com/questions/29075287/ansible-insert-line-if-not-exists](https://stackoverflow.com/questions/29075287/ansible-insert-line-if-not-exists)
    * [https://stackoverflow.com/questions/19688885/in-ansible-how-do-i-add-a-line-to-the-end-of-a-file](https://stackoverflow.com/questions/19688885/in-ansible-how-do-i-add-a-line-to-the-end-of-a-file)
    * [http://www.mydailytutorials.com/ansible-add-line-to-file/](http://www.mydailytutorials.com/ansible-add-line-to-file/)

## Task Modules

* Documentation:

    * module index: [https://docs.ansible.com/ansible/latest/modules/modules_by_category.html](https://docs.ansible.com/ansible/latest/modules/modules_by_category.html)
    * network modules: [https://docs.ansible.com/ansible/latest/modules/list_of_net_tools_modules.html](https://docs.ansible.com/ansible/latest/modules/list_of_net_tools_modules.html)

### apache2_module module

* Documentation: [http://docs.ansible.com/ansible/latest/modules/apache2_module_module.html](http://docs.ansible.com/ansible/latest/modules/apache2_module_module.html)
* Examples:

    * [https://thornelabs.net/2014/06/02/manage-apache-virtualhosts-and-mod-rewrite-rules-with-ansible.html](https://thornelabs.net/2014/06/02/manage-apache-virtualhosts-and-mod-rewrite-rules-with-ansible.html)

* Lets you enable and disable installed apache modules.
* Example: mod_wsgi

        # Enable mod_wsgi
        - name: Enable mod_wsgi module.
          become: yes
          become_user: root
          apache2_module:
            state: present
            name: wsgi

### apt module

* Documentation: [https://docs.ansible.com/ansible/latest/modules/apt_module.html](https://docs.ansible.com/ansible/latest/modules/apt_module.html)

    * Install one:

            - name: install dams
              become: yes
              become_user: root
              apt:
                name: dkms
                state: present

    * Install multiple:

            - name: install OpenSSH
              become: yes
              become_user: root
              apt:
            name:
              - openssh-client
              - openssh-server
              - openssh-sftp-server
            state: present

    * Update cache:

            # update cache
            - name: Update apt cache
              become: yes
              become_user: root
              apt:
                update_cache: yes
                
    * Dist upgrade:

            # upgrade
            - name: Upgrade installed packages
              become: yes
              become_user: root
              apt:
                upgrade: dist

### apt_key module

* Documentation: [https://docs.ansible.com/ansible/latest/modules/apt_key_module.html](https://docs.ansible.com/ansible/latest/modules/apt_key_module.html)

### copy module

* Copy files.
* Documentation: [https://docs.ansible.com/ansible/latest/modules/copy_module.html#copy-module](https://docs.ansible.com/ansible/latest/modules/copy_module.html#copy-module)

### django_manage module

- Documentation:

    - [https://docs.ansible.com/ansible/latest/modules/django_manage_module.html](https://docs.ansible.com/ansible/latest/modules/django_manage_module.html)
    - ansible bug - django_manage createsuperuser does not seem idempotent: [django_manage createsuperuser does not seem idempotent](django_manage createsuperuser does not seem idempotent)
    - * gist - only create superuser if doesn't already exist: [https://gist.github.com/elleryq/9c70e08b1b2cecc636d6](https://gist.github.com/elleryq/9c70e08b1b2cecc636d6)

        - includes better way in last comment.

    - stackoverflow superuser create: [https://stackoverflow.com/questions/45915174/how-to-create-django-superuser-in-ansible-in-idempotent-way](https://stackoverflow.com/questions/45915174/how-to-create-django-superuser-in-ansible-in-idempotent-way)

- Examples:

    - idempotent (only if doesn't already exist) superuser create:

        - [https://gist.github.com/elleryq/9c70e08b1b2cecc636d6](https://gist.github.com/elleryq/9c70e08b1b2cecc636d6)
        - and, see the `django_project` role in [https://github.com/jonathanmorgan/ansible-patterns](https://github.com/jonathanmorgan/ansible-patterns).

                # check if superuser user exists, if not, create user.
                - name: Check if superuser user exists, if not, create user.
                  django_manage:
                    command: shell -c "from django.contrib.auth import get_user_model; MyUser = get_user_model(); MyUser.objects.filter( username__exact = '{{ django_superuser_username }}' ).count() == 0 or exit(); new_super_user = MyUser( username = '{{ django_superuser_username }}', password='{{ django_superuser_password }}', email='{{ django_superuser_email }}', is_superuser = True, is_staff = True ); new_super_user.save();"
                    app_path: "{{ django_project_folder_path }}"
                    virtualenv: "{{ django_virtualenv_path }}"

    - migrate without `django_manage`:

            # run `python manage.py migrate`
            - name: run `python manage.py migrate`
              shell: "{{ django_virtualenv_bin_folder }}/python manage.py migrate"
              args:
                chdir: "{{ django_project_folder }}"

    - migrate with `django_manage`:

            - name: run `python manage.py migrate`
              django_manage:
                app_path: "{{ django_project_folder_path }}"
                command: migrate
                virtualenv: "{{ django_virtualenv_path }}"

### file module

* Used to create, alter, and remove files and directories.
* Documentation: [https://docs.ansible.com/ansible/latest/modules/file_module.html](https://docs.ansible.com/ansible/latest/modules/file_module.html)
* Examples:

    * [http://www.mydailytutorials.com/ansible-create-directory/](http://www.mydailytutorials.com/ansible-create-directory/)
    * `touch` a file:

            # create django log file
            - name: make sure there is a "{{ django_log_file_path }}" log file.
            file:
              path: "{{ django_log_file_path }}"
              state: touch
              mode: 0666

### get_url module

* Documentation: [http://docs.ansible.com/ansible/latest/modules/get_url_module.html](http://docs.ansible.com/ansible/latest/modules/get_url_module.html)

### git module

* Used to interact with git repositories.
* Documentation: [https://docs.ansible.com/ansible/latest/modules/git_module.html](https://docs.ansible.com/ansible/latest/modules/git_module.html)
* Examples:

    * `clone` a repository:

            # prerequisite: python_utilities
            # - https://github.com/jonathanmorgan/python_utilities.git
            # - git@github.com:jonathanmorgan/python_utilities.git
            - name: clone python_utilities
              git:
                repo: https://github.com/jonathanmorgan/python_utilities.git
                dest: "{{ django_project_folder_path }}/python_utilities"

### lineinfile module

* Documentation: [http://docs.ansible.com/ansible/latest/modules/lineinfile_module.html](http://docs.ansible.com/ansible/latest/modules/lineinfile_module.html)
* Examples:
    * append to end of file: [https://stackoverflow.com/questions/19688885/in-ansible-how-do-i-add-a-line-to-the-end-of-a-file](https://stackoverflow.com/questions/19688885/in-ansible-how-do-i-add-a-line-to-the-end-of-a-file)
    * [http://www.mydailytutorials.com/ansible-add-line-to-file/](http://www.mydailytutorials.com/ansible-add-line-to-file/)
    * update matching row within a file (using backrefs and regex):
        * syntax:

            * [https://stackoverflow.com/a/40789373/2237054](https://stackoverflow.com/a/40789373/2237054)
            * [http://daveops.co.uk/2014/05/using-ansible-to-append-a-string-to-the-end-of-a-line/](http://daveops.co.uk/2014/05/using-ansible-to-append-a-string-to-the-end-of-a-line/)

        * regular expression: [https://stackoverflow.com/a/31465939/2237054](https://stackoverflow.com/a/31465939/2237054)
        * add or append: [https://stackoverflow.com/questions/39834862/ansible-lineinfile-add-new-line-with-path-or-append-to-existing-line-with](https://stackoverflow.com/questions/39834862/ansible-lineinfile-add-new-line-with-path-or-append-to-existing-line-with)
        * Example adding host to 127.0.0.1 line in /etc/hosts:

                # add to end of 127.0.0.1 line in /etc/hosts
                - name: Add host name ( {{ server_host_name }} ) to /etc/hosts.
                  become: yes
                  become_user: root
                  lineinfile:
                    dest: /etc/hosts
                    backrefs: yes
                    regexp: '^(127\.0\.0\.1(?!.*\b{{ server_host_name }}\b).*)'
                    line: '\1\t{{ server_host_name }}'

### mysql_db module

* Documentation: https://docs.ansible.com/ansible/latest/modules/mysql_db_module.html#mysql-db-module
* Example:

    * remove test database:

            # remove test database.
            - name: Removes the test database
              mysql_db:
                login_user: root
                login_password: "{{ server_mariadb_root_password }}"
                db: test
                state: absent
              notify: restart mysql
              tags: mariadb

### mysql_user module

* Documentation: https://docs.ansible.com/ansible/latest/modules/mysql_user_module.html#mysql-user-module
* Example:

    * delete all anonymous users (all hosts):

            # Delete anonymous users
            - name: Deletes anonymous server user
              mysql_user:
                login_user: root
                login_password: "{{ server_mariadb_root_password }}"
                name: ""
                host_all: yes
                state: absent
              notify: restart mysql
              tags: mariadb

    * create new non-root admin user:

            # create non-root admin user?
            # Creates database user {{ server_mariadb_admin_user }} and password {{ server_mariadb_admin_password }} with all database privileges and 'WITH GRANT OPTION'
            - name: Create non-root admin user {{ server_mariadb_admin_user }}
              mysql_user:
                login_user: root
                login_password: "{{ server_mariadb_root_password }}"
                name: "{{ server_mariadb_admin_user }}"
                password: "{{ server_mariadb_admin_password }}"
                host: "{{ item }}"
                priv: '*.*:ALL,GRANT'
                state: present
              with_items:
                - "%"
                - ::1
                - 127.0.0.1
                - localhost
              when:
                - server_mariadb_admin_user != ""
                - server_mariadb_admin_password != ""
              notify: restart mysql
              tags: mariadb

### npm module

* Documentation: [https://docs.ansible.com/ansible/latest/modules/npm_module.html](https://docs.ansible.com/ansible/latest/modules/npm_module.html)

### openssl_* modules

* Documentation:

    * [https://docs.ansible.com/ansible/latest/modules/list_of_crypto_modules.html](https://docs.ansible.com/ansible/latest/modules/list_of_crypto_modules.html)
    * openssl_certificate: [https://docs.ansible.com/ansible/latest/modules/openssl_certificate_module.html](https://docs.ansible.com/ansible/latest/modules/openssl_certificate_module.html)
    * openssl_csr: [https://docs.ansible.com/ansible/latest/modules/openssl_csr_module.html](https://docs.ansible.com/ansible/latest/modules/openssl_csr_module.html)
    * openssl_dhparam: [https://docs.ansible.com/ansible/latest/modules/openssl_dhparam_module.html](https://docs.ansible.com/ansible/latest/modules/openssl_dhparam_module.html)
    * openssl_privatekey: [https://docs.ansible.com/ansible/latest/modules/openssl_privatekey_module.html](https://docs.ansible.com/ansible/latest/modules/openssl_privatekey_module.html)
    * openssl_publickey: [https://docs.ansible.com/ansible/latest/modules/openssl_publickey_module.html](https://docs.ansible.com/ansible/latest/modules/openssl_publickey_module.html)

* Examples:

    * create self-signed certificate:

        * From: [https://www.jeffgeerling.com/blog/2017/generating-self-signed-openssl-certs-ansible-24s-crypto-modules](https://www.jeffgeerling.com/blog/2017/generating-self-signed-openssl-certs-ansible-24s-crypto-modules)
        * 1) make private key with `openssl_privatekey`.
        * 2) generate CSR with `openssl_csr`.
        * 3) generate certificate using `openssl_certificate`.

    * To create a self-signed certificate (jupyterhub example):

            # generate OpenSSL private key.
            - name: Generate an OpenSSL private key.
              become: yes
              become_user: root
              openssl_privatekey:
                path: "{{ jupyterhub_config_folder }}/jupyterhub_key.pem"
            # create certificate signing request (CSR)
            - name: Generate an OpenSSL CSR.
              become: yes
              become_user: root
              openssl_csr:
                path: "{{ jupyterhub_config_folder }}/jupyterhub.csr"
                privatekey_path: "{{ jupyterhub_config_folder }}/jupyterhub_key.pem"
                common_name: "{{ server_name }}"
            # generate certificate
            - name: Generate a Self-Signed OpenSSL certificate.
              become: yes
              become_user: root
              openssl_certificate:
                path: "{{ jupyterhub_config_folder }}/jupyterhub_cert.pem"
                privatekey_path: "{{ jupyterhub_config_folder }}/jupyterhub_key.pem"
                csr_path: "{{ jupyterhub_config_folder }}/jupyterhub.csr"
                provider: selfsigned

### pip module

* Documentation: [http://docs.ansible.com/ansible/latest/modules/pip_module.html](http://docs.ansible.com/ansible/latest/modules/pip_module.html)
* Examples:
    * [https://stackoverflow.com/questions/31396130/ansible-install-multiple-python-packages-on-a-single-session](https://stackoverflow.com/questions/31396130/ansible-install-multiple-python-packages-on-a-single-session)
* can run a particular pip executable
    * under pip, include `executable: <path_to_pip>`.  This could be just `pip3`, or could be something like `/usr/bin/pip3`.
    * Example:

            # ==> virtualenv and virtualenvwrapper
            - name: install virtualenv and virtualenvwrapper
              become: yes
              become_user: root
              pip:
                executable: pip3
                name:
                  - virtualenv
                  - virtualenvwrapper
* supports virtualenvs
    * under pip, include the “virtualenv: <path_to_virtualenv_folder>”.
    * If virtualenv not initialized, will first initialize at path you specify.
    * Example:

            # Install specified python requirements in indicated (virtualenv).
            - pip:
                requirements: /my_app/requirements.txt
                virtualenv: /my_app/venv

### postgresql_db module

- Documentation: [https://docs.ansible.com/ansible/latest/modules/postgresql_db_module.html](https://docs.ansible.com/ansible/latest/modules/postgresql_db_module.html)
- Examples:

    - create with login username and password (to get around oddness with using postgres user):

            # create database for django app in postgres
            - name: create postgresql database for django_project named {{ django_db_name }}
              postgresql_db
                login_user: "{{ server_pgsql_admin_user }}"
                login_password: "{{ server_pgsql_admin_password }}"
                db: postgres
                name: "{{ django_db_name }}"
                owner: "{{ django_db_username }}"

### postgresql_user module

* Documentation: [https://docs.ansible.com/ansible/latest/modules/postgresql_user_module.html](https://docs.ansible.com/ansible/latest/modules/postgresql_user_module.html)
* Notes:

    * requires the psycopg2 module be installed in the python environment that ansible is using to do install tasks.
    * requires encrypted passwords in PostgreSQL 10 - not sure of implications of this.

* Errors:

    * on ubuntu, if you get error: FAILED! => {"changed": false, "msg": "unable to connect to database: FATAL: Peer authentication failed for user \"postgres\"\n"}

        * try to install package “acl” on ubuntu and try again.  Didn’t work for me.
        * for this (no clue why):

                - name: Create non-postgres admin user
                  become: yes
                  become_user: postgres
                  postgresql_user:
                    name: '{{server_pgsql_admin_user}}'
                    password: '{{server_pgsql_admin_password}}'
                    role_attr_flags: SUPERUSER
                    encrypted: yes
                  when:
                    - server_pgsql_admin_user != ""
                    - server_pgsql_admin_password != ""
                            * Alternative (not pretty, but it works):

                # create non-postgres admin user?
                - name: Create non-postgres admin user
                  become: yes
                  become_user: root
                  shell: sudo -u postgres psql -c "DO \$do\$ BEGIN IF NOT EXISTS( SELECT * FROM pg_catalog.pg_roles WHERE rolname = '{{server_pgsql_admin_user}}') THEN CREATE USER {{server_pgsql_admin_user}} WITH PASSWORD '{{server_pgsql_admin_password}}' SUPERUSER; END IF; END \$do\$;"
                  when:
                    - server_pgsql_admin_user != ""
                    - server_pgsql_admin_password != “"

### service module

* Documentation: [https://docs.ansible.com/ansible/latest/modules/service_module.html](https://docs.ansible.com/ansible/latest/modules/service_module.html)
* Notes:

    * under the hood, for systemd systems, implemented using systemd module: [https://docs.ansible.com/ansible/latest/modules/systemd_module.html](https://docs.ansible.com/ansible/latest/modules/systemd_module.html)

* Examples:

    * Enable new systems service:

            # Create systemd service for jupyterhub.
            - name: Create systemd service for jupyterhub
              become: yes
              become_user: root
              template:
                src: jupyterhub.service.j2
                dest: /etc/systemd/system/jupyterhub.service
            # Enable jupyterhub service - have to say use “service”, systemd service has bug.
            # https://github.com/ansible/ansible-modules-core/issues/3764
            - name: Enable jupyterhub service
              become: yes
              become_user: root
              service:
                name: jupyterhub
                enabled: True
                use: service
            # Start jupyterhub and set as a startup service
            - name: Enable jupyterhub as a startup service
              become: yes
              become_user: root
              service:
                name: jupyterhub
                state: started
                enabled: yes
                    * Restart Apache:
            # restart apache
            - name: restart apache2
              become: yes
              become_user: root
              service:
                name: apache2
                state: restarted

### shell module

* Documentation: [https://docs.ansible.com/ansible/latest/modules/shell_module.html](https://docs.ansible.com/ansible/latest/modules/shell_module.html)
* execute raw shell commands.

### stat module

* Lets you check the state of file or directory on the file system.
* Documentation: [https://docs.ansible.com/ansible/latest/modules/stat_module.html](https://docs.ansible.com/ansible/latest/modules/stat_module.html)
* Notes:
* Examples:
    * look for file /etc/foo.conf, store details in variable named “st”:

            - name: check if /etc/foo.conf exists
              stat:
                path: /etc/foo.conf
                register: st

### template module

* Documentation: [https://docs.ansible.com/ansible/latest/modules/template_module.html](https://docs.ansible.com/ansible/latest/modules/template_module.html)
* Examples:

    * CRAN R apt source:

            # Add apt source for CRAN R.
            - name: Create APT source for CRAN R.
              template:
                src: CRAN-R.list.j2
                dest: /etc/apt/sources.list.d

### ufw module

* Documentation: [https://docs.ansible.com/ansible/latest/modules/ufw_module.html](https://docs.ansible.com/ansible/latest/modules/ufw_module.html)
* Examples:
    * [https://github.com/Oefenweb/ansible-ufw](https://github.com/Oefenweb/ansible-ufw)

### uri module

* Documentation - [http://docs.ansible.com/ansible/latest/modules/uri_module.html](http://docs.ansible.com/ansible/latest/modules/uri_module.html)

# ansible modules:

* from [https://stackoverflow.com/questions/42836598/how-to-download-and-install-ansible-modules](https://stackoverflow.com/questions/42836598/how-to-download-and-install-ansible-modules):

    * An excellent general guide to writing modules (I've no connection to the author) can be found here: [http://blog.toast38coza.me/custom-ansible-module-hello-world/](http://blog.toast38coza.me/custom-ansible-module-hello-world/)
    * The quickest way is to simply have a folder called library/ in the same folder as your playbook. Inside this folder, place the python script for the Ansible Module. You should now have a corresponding task available to your playbook.
    * If you want to share your module across multiple projects, then you can add an entry to `/etc/ansible/ansible.cfg` pointing to a shared library location, eg:

        * `library        = /usr/share/ansible/library`

# TODO

todo:

- jupyter kernel-ize research virtualenv.
- git config
- set up one or more standard users, including git config, Downloads folder, etc.
