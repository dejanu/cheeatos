## Cheatsheet collection

* [GCP](index.md)

### ANSIBLE

## AD-HOC

```bash
ansbile <GROUP> -m shell -a "uptime" -i <HOSTS_INVENTORY>

# transfer file  (sudo yum install -y libselinux-python )
ansible <GROUP> -m copy -a "src=/path/to/file dest=/path/on/target/" -i <HOSTS_INVENTORY>
```
 

## Inventory

```bash
# ansible view inventory
ansible-inventory -i <HOSTS_INVENTORY> --graph
```
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
```
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