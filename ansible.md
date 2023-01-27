## Cheatsheet collection

* [Home](index.md)
* <ins>[Ansible](ansible.md)</ins>
* [Git](git.md)
* [GCP](gcp.md)
* [Docker](docker.md)
* [Azure](azure.md)
* [Terraform](terraform.md)
* [Helm](helm.md)
* [ElasticSearch](elastic.md)
* [Kubernetes](k8s.md)
* [Istio](istio.md)
* [OIDC](openID.md)
* [Workshop](workshop.md)

---

### ANSIBLE

* Agentless (uses as transport layer Open SSH)
* Configuration (`/etc/ansible/ansible.cfg`) check: `$ansible-config list`
* Ansible inventory commands: `$ansible-inventory --list` or `$ansible-inventory --graph`

## Ansible configuration

*  Is defined in `anisble.cfg` in the current directory or `~/.ansible.cfg` or `/etc/ansible/ansible.cfg`:

```yml
[defaults]
# define default inventory to bypass -i usage
inventory = hosts.yml

# gives freedom of not specifying the encryption password or the password file every time
vault_password_file = ~/.vault_pass

retry_files_enabled = false
hash_behaviour=merge
forks = 10
## OpenSSH needed to use jumphost. See hosts.yml
transport = ssh
merge_multiple_cli_tags=True
# Use the YAML callback plugin.
stdout_callback = yaml

#interpreter discovery
interpreter_python = /usr/bin/python3
```

* Use `ansible_python_interpreter` inventory variable in `hosts.yml` or the `interpreter_python` key in the `[defaults]` section of `ansible.cfg` to set the desired python interpreter. 



## AD-HOC


* Can be used to reboot servers, copy files, manage packages and users etc: `ansible <HOST_GROUP> -m <MODULE> -a "ARGUMENTS"`

```bash

# ping 
ansible <GROUP> -v -m ping -i <HOSTS_INVENTORY>
ansbile <GROUP> -m shell -a "uptime" -i <HOSTS_INVENTORY>

# transfer file  (sudo yum install -y libselinux-python )
ansible <GROUP> -m copy -a "src=/path/to/file dest=/path/on/target/" -i <HOSTS_INVENTORY>

```
 

## Inventory

```bash
# ansible view inventory
ansible-inventory -i <HOSTS_INVENTORY> --graph
```
* Run playbook on local host: `ansible-playbook --connection=local <PLAYBOOK>.yml`:

```bash
# ansible-playbook run play on localhost
-

  name: playbook for localhost

  hosts: localhost

  connection: local
```

***

## Tags

```bash
# ansible-playbook list tags

ansible-playbook -i <HOSTS_INVENTORY> <PLAYBOOK.yml> --list-tags
ansible-playbook -i <HOSTS_INVENTORY> <PLAYBOOK.yml> -t <TAG_NAME>
```
```bash
# ansible playbook run with tags
ansible-playbook -l <HOSTNAME/GROUP> -i <HOSTS_INVENTORY> <PLAYBOOK.yml> -t "<TAG_NAME>"
ansible-playbook -l <HOSTNAME/GROUP> -i <HOSTS_INVENTORY> --tags "selected-tag" -u root <PLAYBOOK>.yml` 
```
* Also we can use `--skip-tags` flag  
*

***

## Extra Vars

```bash
# ansible playbook with --extra-vars

ansible-playbook -i <HOSTS_INVENTORY> <PLAYBOOK.yml> -e "instance_name=search_term"
```
 
```yaml
# playbook with vars

- name: "Play with var as cmd argument"

      shell: grep -l {{instance_name}}


# use env variables

- name: "Debug"

    debug:

      msg: "{{ lookup('env','HOME') }} value for HOME var"

 

  - name: "Get api key from file"

    shell: "cat /home/{{ ansible_env.USER }}/name "

    register: api_key

    args:

      executable: /bin/sh

```
***

## Test/verify/lint playbooks

* Fork the linter: https://github.com/adrienverge/yamllint. 

1) Dry Run using check module: `ansible-playbook <PLAYBOOK>.yml --check` or just syntax check it `ansible-module <PLAYBOOK>.yml --syntax-check`

2) Linter: ansible molecule to test your ansible roles: `pip install molecule`  

3) List playbook tags: `ansible-playbook --list-tags <PLAYBOOK>.yml`

***

## Ansible Vault

* Ansible Vault encrypts variables and files:
* Ansible vault file: `variables_encrypted.yml`:

```bash
#environment variable to specify that file
echo $ANSIBLE_VAULT_PASSWORD_FILE

# vault file for sensitive data
variables_encrypted.yml

# encrypt/devrypt vault file
ansible-vault encrypt variables_encrypted.yml --encrypt-vault-id default
ansible-vault decrypt variables_encrypted.yml --encrypt-vault-id default
ansible-vault view variables_encrypted.yml
```

## Variables

* Inventory variables

```bash
# Ansible <2.0:
[all:vars]
ansible_connection=ssh
ansible_ssh_user=vagrant 
ansible_ssh_pass=vagrant

# Ansible >=2.0:
[all:vars]
ansible_connection=ssh # actually default mode smart is OK
ansible_user=vagrant
ansible_pass=vagrant # or ansible_ssh_pass=vagrant

```

* Print/register var/output

```yml
- name: Test connection
  #hosts: "prometheus_host"
  hosts: "grafana_host"
  gather_facts: no
  tasks:
    - name: Check user
      shell: id
      register: shellcmd
        
    - debug: msg="{{shellcmd.stdout}}"

    # - debug:
    #     var: shellcmd
```
---

```bash
                    ___ _____
                   /\ (_)    \
                  /  \      (_,
                 _)  _\   _    \
                /   (_)\_( )____\
                \_     /    _  _/
                  ) /\/  _ (o)(
                  \ \_) (o)   /
                   \/________/         @dejanualex
```