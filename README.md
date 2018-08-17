# Ansible RefCard
A very usefull RefCard for Ansible usage.
Written by Germain LEFEBVRE by August 2018 from Ansible v2.5 usage.


**Table of Contents**
1. [Context](#context)
2. [Ansible Definitions](#ansible-definitions)
3. [Ansible AdHoc Commands](#ansible-adhoc-commands)
4. [Ansible Inventories](#ansible-inventories)
5. [Ansible Tasks](#ansible-tasks)
   1. [Editing files](#editing-files)
   2. [Archiving](#archiving)
   3. [Manage services](#manage-services)
   4. [Run linux commands](#run-linux-commands)
   5. [Interact with webservices](#interact-with-webservices)
6. [Ansible Playbooks](#ansible-playbooks)
7. [Ansible Variables](#ansible-variables)
8. [Ansible Roles](#ansible-roles)
9. [Ansible Modules](#ansible-modules)

   

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


## Ansible Definitions

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

#### Inventory with a single server (not very useful) :
```
server.domain.fr
```

#### Inventory with range servers (server01 to server09) :
```
server[01-09].domain.fr
```

#### Inventory with multiple servers :
```
rhel01.domain.fr
deb01.domain.fr
```

#### Inventory with aliases :
```
rhel01 rhel01.domain.fr
deb01 deb01.domain.fr
```

#### Inventory with groups of servers :
```
rhel01 rhel01.domain.fr
deb01 deb01.domain.fr

[redhat]
rhel01

[debian]
deb01
```

#### Inventory with groups of groups (a concrete example) that you name `inventories/servers` :
```
rhel01 rhel01.domain.fr
deb01 deb01.domain.fr
win01 win01.domain.fr

[redhat]
rhel01

[debian]
deb01

[linux:children]
redhat
debian

[windows]
win01
```


## Ansible Tasks

These are a few examples of Ansible Tasks simply

### Manage files

#### Create a file
```
- file:
    path: /etc/file
    state: touch
```

#### Set ownership and rights on file
```
- file:
    path: /etc/file
    owner: root
    group: root
    mode: "0740"
```

#### Set ownership and rights on directory on all its children
```
- file:
    path: /etc/dir/
    owner: root
    group: root
    mode: "0740"
    recurse: yes
```

#### Set symlink
```
- file:
    src: /file/to/link/to
    dest: /path/to/symlink
    state: link
```

#### Copy local file to remote server
```
- copy:
    src: /local/server/file
    dest: /remote/server/file
    owner: root
    group: root
    mode: 0640
```

#### Copy files from serverA to serverB
Server A and server B are defined in the inventory. Server B is the targeted host for the task.
```
- copy:
    src: /remote/serverA/dir/file
    dest: /remote/serverB/dir/file
  delegate_to: serverA
```

#### Copy current dir file to remote server
Ansible will take common path as current running command dir (where tasks or playbook are)
```
- copy:
    src: file
    dest: /remote/server/file
```

#### Copy remote file to another path on the same host
```
- name: Copy a "sudoers" file on the remote machine for editing
  copy:
    src: /etc/file
    dest: /etc/copy
    remote_src: yes
```

#### Generate a file from a template (using variables)
```
- template:
    src: /mytemplates/foo.j2
    dest: /etc/file.conf
```

#### Update sshd configuration safely, avoid locking yourself out
Copy Jinja2 template from surrent dir `etc/ssh/sshd_config.j2` to remote and ensure you are not stuck in the middle.
```
- template:
    src: etc/ssh/sshd_config.j2
    dest: /etc/ssh/sshd_config
    owner: root
    group: root
    mode: '0600'
    validate: /usr/sbin/sshd -t -f %s
    backup: yes
```

### Editing files

#### Use sed tool to change value
```
- lineinfile:
    path: /etc/selinux/config
    regexp: '^SELINUX='
    line: 'SELINUX=enforcing'
```

#### Ensure line is not present in file
```
- lineinfile:
    path: /etc/sudoers
    state: absent
    regexp: '^%wheel'
```

#### Insert line after pattern
```
- lineinfile:
    path: /etc/httpd/conf/httpd.conf
    regexp: '^Listen '
    insertafter: '^#Listen '
    line: 'Listen 8080'
```

#### Insert line before pattern
```
- lineinfile:
    path: /etc/services
    regexp: '^# port for http'
    insertbefore: '^www.*80/tcp'
    line: '# port for http by default'
```

#### Insert block lines (and changing Marker Pattern)
```
- blockinfile:
    path: /vetc/hosts
    marker: "### ANSIBLE MANAGED BLOCK ###"
    insertafter: "localhost"
    content: |
      192.168.0.10 printer01
      192.168.0.11  printer02
```

#### Replace pattern usgin sed tool
```
- replace:
    path: /etc/hosts
    regexp: '(localhost)(.*)$'
    replace: '\1localdomain \1\2'
    backup: yes
```

### Archiving

#### Copy a tarball on remote and unarchive it
```
- name: Extract foo.tgz into /var/lib/foo
  unarchive:
    src: foo.tgz
    dest: /var/lib/foo
```

#### Untar a archive already on remote server
```
- unarchive:
    src: /tmp/foo.zip
    dest: /usr/local/bin
    remote_src: yes
```

#### Download tar from link and unarchive it
```
  unarchive:
    src: https://domain.fr/archive.zip
    dest: /usr/local/bin
    remote_src: yes
```

### Manage services

#### Restart service
```
- service: # for rhel6 or systemd: for rhel7
    name: httpd
    state: restart
```

### Run linux commands

#### Run single command
```
- command: cat /etc/hosts
```

#### Run script
```
- shell: /var/script/reboot.sh
```

#### Run multiline commands
```
- shell: |
    echo "toto"
    exit 0
```

### Interact with webservices

#### Check a 200 response for a GET request
```
- uri:
    url: http://www.example.com
```

#### Send a body in POST request with an Basic Auth
Send an issue in body to a secured Jira and ensure it was gigested (201 response code)
```
- uri:
    url: https://your.jira.example.com/rest/api/2/issue/
    method: POST
    user: your_username
    password: your_pass
    body: "{{ lookup('file','issue.json') }}"
    force_basic_auth: yes
    status_code: 201
    body_format: json
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
redhat
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

#### Install yum package _httpd_ on RedHat servers

`vi httpd.yml`
```
---
- hosts: [redhat]
  become: true
  tasks
  - yum:
      name: httpd
      state: installed
```

Run playbook on inventory `servers`

`ansible-playbook -i inventory/servers httpd.yml`




## Ansible Variables

Let's start with a hosrt but consise inventory.

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
redhat
debian

[windows]
win01
```

#### Write variables for all the servers whatever the inventory

Variables

`vi group_vars/all`
```
timezone: 'Europe/Paris'
```

Playbook

``
```
---
- hosts: [linux]
  become: true
  tasks:
  - name: Change server timezone
    file:
      src: /usr/share/zoneinfo/{{ timezone }}
      dest: /etc/localtime
      state: link
      force: yes
```



## Ansible Roles

**Role directories**
```
roles/
└── my-role
    ├── defaults
    │   └── main.yml
    ├── files
    ├── handlers
    │   └── main.yml
    ├── tasks
    │   └── main.yml
    ├── templates
    └── vars
        └── main.yml
```

**Call role in your playbook**
```
- hosts: all
  roles:
  - my-role
```

#### Role with simple task

```
- command: cat /etc/hosts
```

#### Role with file to copy
File at `roles/my-role/files/my-file.sh`

Task at `roles/my-role/tasks/main.yml`
```
- copy:
    src: my-file.sh
    dest: /tmp/my-file.sh
```

#### Role with template to generate
Template at `roels/my-role/templates/my-template.sh.j2`
```
#!/bin/bash
echo "{{ timezone }}"
```

Task at `roles/my-role/tasks/main.yml`
```
- template:
    src: my-template.sh.j2
    dest: /tmp/my-template.sh
```

#### Role with trigger to handler
Handler at `roles/my-role/handlers/main.yml`
```
- name: Restart Apache
  systemd:
    name: httpd
    state: restart
```

Task at `roles/my-role/tasks/main.yml`
```
- copy:
    src: httpd.conf
    dest: /etc/httpd/conf/httpd.conf
  notify: Restart Apache
```
Will run the handler `Restart Apache` if task `copy` state has changed.

#### Role with default variables
Default variables at ``
```
apache_version: '2.4.2'
```

Task at `roles/my-role/tasks/main.yml`
```
- yum:
    name: httpd-{{ apache_version }}
    state: present
```


## Including and importing playbooks or tasks or roles

### Include tasks
Both `import_task` and `include_task`work.
```
- hosts: [redhat]
  tasks:
  - import_tasks: roles/my-role/tasks/main.yml
  - include_tasks: roles/my-role/tasks/main.yml
```

### Include tasks but filter on tag in ansible-playbook command
Only `import_tasks` works and evaluates tags from tasks.

### Include playbook
Only `import_playbook` works for including a whole playbook in another one.
```
- import_playbook: install_apache.yml
```

### Include a whole role
Both `import_role` and `include_role` can be used to call tasks from a role.
```
- hosts: [redhat]
  tasks:
  - import_role:
      name: my-role
      tasks_from: main

```

### Include a taskfile from a role
Permit to load all variables inherant to the role.
```
- hosts: [redhat]
  tasks:
  - include_role:
      name: my-role
      tasks_from: install # Will call roles/my-role/tasks/install.yml
```

### Include tasks but filter on tag in ansible-playbook command
Only `import_role` works for including a whole role in a playbook using tags on commands.
```
- hosts: [redhat]
  tasks:
  - import_role:
      name: my-role
```


## Ansible Modules


