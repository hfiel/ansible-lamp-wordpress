# ansible-lamp-wordpress

Files for the OpenWebinars ansible course: https://openwebinars.net/cursos/ansible/
Forked from: https://github.com/MarioPerezEsteso/ansible-lamp-wordpress

Modifications for Ubuntu 18 by hfiel

## Project: wordpress server in Ubuntu 18 using ansible with 4 nodes
* 1 LB with HAProxy
* 2 WEB nodes with Apache and Wordpress
* 1 DB node with MySQL

## Notes

### Regarding modifications to content

Compared to the original course content, I have changed IP addresses, several parameter/vars, names and tasks. Don't expect copy/paste/mix from this and the original course files to work 100%.

### Regarding Ubuntu

The original course uses Ubuntu 16. I have used Ubuntu Server 18.04.01 in the 4 nodes, with a clean  installation. Some changes have been required after the clean install.


#### apt
I have modified */etc/apt/sources.list* to allow universe and multiverse packages

ORIGINAL:

```
deb http://archive.ubuntu.com/ubuntu bionic main
```

MODIFIED:

```
deb http://archive.ubuntu.com/ubuntu bionic main universe multiverse
```

#### Users

* The local user can sudo without password (via sudoers, this is explained in the course)
* *root* allows SSH login, even with password (this is explained in the course)

### Regarding ansible

#### Version

Due to some issues with Python3 and the mysql_user module dependencies, I used the ansible version 2.8.0.dev0 (installed from source)

#### Inventory

Ubuntu 18 uses Python 3. In your inventory files you need to define the python intepreter for each node with *ansible_python_interpreter*:

```
node_name ansible_host=aaa.bbb.ccc.ddd ansible_user=root ansible_python_interpreter=/usr/bin/python3
```

#### MySQL

In order to be able to connect to MySQL with the root user, two tasks are required before the apt install:

(see https://stackoverflow.com/questions/40050219/rootlocalhost-password-set-with-ansible-mysql-user-module-doesnt-work#40059130)

```
- name: Specify MySQL root password before installing
  debconf:
    name: 'mysql-server'
    question: 'mysql-server/root_password'
    value: '{{mysql_root_db_pass | quote}}'
    vtype: 'password'
  become: true

- name: Confirm MySQL root password before installing
  debconf:
    name: 'mysql-server'
    question: 'mysql-server/root_password_again'
    value: '{{mysql_root_db_pass | quote}}'
    vtype: 'password'
  become: true
```

#### Additional playbooks

I have created a simple *updateall.yml* playbook to update the apt caches of all the hosts. To use the playbook, put your hosts in the */etc/ansible/hosts* inventory or supply an inventory file with *-i*

```
ansible -i inventory_file updateall.yml
```
