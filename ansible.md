## Cheatsheet collection

* [GCP](index.md)

### ANSIBLE

* Agentless (uses as transport layer Open SSH)
* Configuration (`/etc/ansible/ansible.cfg`) check: `$ansible-config list`
* Ansible inventory commands: `$ansible-inventory --list` or `$ansible-inventory --graph`

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

* Ansible vault file: variables_encrypted.yml:

```bash
#view/decrypt ansible vault 

ansible-vault view variables_encrypted.yml
ansible-vault decrypt/encrypt variables_encrypted.yml
```
