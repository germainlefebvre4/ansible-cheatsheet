# Ansible CheatSheet
A very usefull CheatSheet for Ansible usage.
Written by Germain LEFEBVRE by August 2018 from Ansible v2.5 usage.


**Table of Contents**
1. [Context](#context)
2. [Ansible Definitions](#ansible-definitions)
3. [Ansible Commands](#ansible-adhoc-commands)
   1. [AdHoc Commands](#ansible-adhoc-commands)

### Context


This is the list of the packages version used to make this CheatSheet

Show your Linux version (here is CentOS/RedHat dist.) with `cat /etc/redhat-release` :
```
CentOS Linux release 7.5.1804 (Core)
```

Python version on your system with `python --version` :
```
Python 2.7.5
```

Finally  Ansible version used with `ansible --version` :
```
ansible 2.5.3
  config file = /etc/ansible/ansible.cfg
  configured module search path = [u'/home/ansible/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python2.7/site-packages/ansible
  executable location = /bin/ansible
  python version = 2.7.5 (default, Apr 11 2018, 07:36:10) [GCC 4.8.5 20150623 (Red Hat 4.8.5-28)]
```


## Ansible definitions

**Ansible Facts**

The Facts are variables used by Ansible to persist data through hosts and executions. There are native facts are stored on local server (disk or in-memory) and user facts defined by Human actions. It is important to get that facts are stored by group of actions OR by host.


**Ansible Hosts**

The Hosts are just servers that the Ansible Server can reach trhough its preferred protocol.


**Ansible Inventories**

The Inventories are list of Hosts defined by FQDN and tidied in groups. Groups can be composed with hosts or group of hosts. Hosts can be aliased to make the list customisable.


**Ansible Tasks**

The Tasks are the actions launched on remote Hosts. Tasks are written in YAML langage in a descriptive structure way making the read and write uniform through any tasks in the world.


**Ansible Variables**

The Variables are the way for Ansible to pass custom values in tasks (and more over).


**Ansible Playbooks**

The Playbooks are the gathering of tasks and hosts. So a Playbook defines a list of tasks to perform on remote servers.


**Ansible Roles**

The Roles are the tidy way to write playbooks. It permits to store a group of actions with the same purpose and to call them in playbooks in a single line.


**Ansible Handlers**

Ansible Handlers are action triggers called from tasks and run at the end of a play. A Handler is a tasks defined by its name and called with its name.


**Ansible Modules**

Modules are scripts written in Python and making uniform actions possible in giving common inputs for users and generating outputs regarding the command played in the module.



## Ansible AdHoc Commands

AdHoc commands are commands launched on the fly with writting any complex structured files.
Let's first use Ansible AdHoc commands with local machine.

`ansible --help`
```
Usage: ansible <host-pattern> [options]

Define and run a single task 'playbook' against a set of hosts
```

Ask remote server if connection is alright on the remote server (here localhost) with module `ping` :

`ansible localhost -m ping`


See all Ansible Facts of the server (here localhost) with module `setup` :

`ansible localhost -m setup`


Show a Ansible Variable generated o nthe remote server with module `debug`:

`ansible localhost -m setup -a 'myVar'`


Run a shell command -here `uptime`) on the remote server (here localhost) :

`ansible localhost -m shell -a 'uptime'`


If you use Ansible Inventories this is the way to use the AdHoc commands.
Ask all remote servers from inventory `servers` if connection is alright :

`ansible all -i inventories/servers -m ping`


Like this you can use a large panel of Ansible Modules to make single action on the remote server(s).


## Ansible Inventories

Ansible Inventories make possible to gather servers in a single file so running commands on all these hosts in a single command. File is formated in a custom langage with similarities with INI files.

We usually make a directory to tidy the inventory files.

Inventory with a single server (not very useful) :

`$ vi inventories/server_single`
```
server.domain.fr
```

Inventory with range servers (server01 to server09) :

`$ vi inventories/servers_range`
```
server[01-09].domain.fr
```

Inventory with multiple servers :

`$ vi inventories/servers_multiple`
```
rhel01.domain.fr
deb01.domain.fr
```

Inventory with aliases :

`$ vi inventories/servers_aliases`
```
rhel01 rhel01.domain.fr
deb01 deb01.domain.fr
```

Inventory with groups of servers :

`$ vi inventories/servers_groups`
```
rhel01 rhel01.domain.fr
deb01 deb01.domain.fr

[redhat]
rhel01

[debian]
deb01
```

Inventory with groups of groups (a concrete example) that you name `inventories/servers` :

`$ vi inventories/servers`
```
rhel01 rhel01.domain.fr
deb01 deb01.domain.fr
win01 win01.domain.fr

[redhat]
rhel01

[debian]
deb01

[linux:children]
rhel
debian

[windows]
win01
```


## Ansible Playbooks

Now we know how are formed Tasks and Inventories we can play with Playbooks and start to really run commands on remote servers. Let's take the last previous inventory :

`$ vi inventories/servers`
```
rhel01 rhel01.domain.fr
deb01 deb01.domain.fr
win01 win01.domain.fr

[redhat]
rhel01

[debian]
deb01

[linux:children]
rhel
debian

[windows]
win01
```

#### Show Ansible Facts of all servers of the Inventory

Build your first Ansible Playbook

`vi setup.yml`
```
---
- hosts: all
  tasks:
  - setup:
```

And run your playbook on :

`ansible-playbook -i inventories/servers setup.yml`


#### Run the command `uptime` on linux group servers

`vi uptime.yml`
```
---
- hosts: [linux]
  tasks:
  - shell: 'uptime'
```

Run your playbook on :

`ansible-playbook -i inventories/servers uptime.yml`
