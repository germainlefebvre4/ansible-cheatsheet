# Ansible RefCard
A very usefull RefCard for Ansible usage.
Written by Germain LEFEBVRE by August 2018 from Ansible v2.5 usage.


**Table of Contents**
1. [Context](#context)
1. [Upgrade your Ansible version](#upgrade-your-anisble-version)
1. [Ansible Definitions](#ansible-definitions)
1. [Ansible AdHoc Commands](#ansible-adhoc-commands)
1. [Ansible Inventories](#ansible-inventories)
1. [Ansible Tasks](#ansible-tasks)
   1. [Editing files](#editing-files)
   1. [Archiving](#archiving)
   1. [Manage services](#manage-services)
   1. [Run linux commands](#run-linux-commands)
   1. [Interact with webservices](#interact-with-webservices)
1. [Ansible Playbooks](#ansible-playbooks)
1. [Ansible Variables](#ansible-variables)
   1. [Call a variable](#call-a-variable)
   1. [Define a variable](#define-a-variable)
   1. [Variable precedence](#variable-precedence)
1. [Ansible Plays](#ansible-plays)
   1. [Available attributes](#available-attributes)
1. [Ansible Roles](#ansible-roles)
   1. [Structure of a role](#structure-of-a-role)
1. [Ansible Modules](#ansible-modules)

   

### Context


This is the list of the packages version used to make this RefCard

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


## Upgrade your Ansible version

Ansible give their Roadmap for v2.5 : [https://docs.ansible.com/ansible/2.5/roadmap/ROADMAP_2_5.html](https://docs.ansible.com/ansible/2.5/roadmap/ROADMAP_2_5.html)


Anisble provides porting guides to help you keeping up-to-date:
* [Ansible 2.0 Porting Guide](https://docs.ansible.com/ansible/2.5/porting_guides/porting_guide_2.0.html)
* [Ansible 2.3 Porting Guide](https://docs.ansible.com/ansible/2.5/porting_guides/porting_guide_2.3.html)
* [Ansible 2.4 Porting Guide](https://docs.ansible.com/ansible/2.5/porting_guides/porting_guide_2.4.html)
* [Ansible 2.5 Porting Guide](https://docs.ansible.com/ansible/2.5/porting_guides/porting_guide_2.5.html)


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

Test connection sith remote host:

`ansible localhost -m ping`


See all Ansible Facts:

`ansible localhost -m setup`


Resolve a var `myVar`:

`ansible localhost -m debug -a 'myVar'`


Run a shell command on remote server:

`ansible localhost -m shell -a 'uptime'`


Apply a task on an inventory `inventories/servers`:

`ansible all -i inventories/servers -m ping`



## Ansible Inventories

Ansible Inventories make possible to gather servers in a single file so running commands on all these hosts in a single command. File is formated in a custom langage with similarities with INI files.

We usually make a directory to tidy the inventory files.

### A bunch of practice examples :
```
server.domain.fr
server[01-09].domain.fr

rhel01 rhel01.domain.fr
deb01 deb01.domain.fr
arch01 arch01.domain.fr
win01 win01.domain.fr

[redhat]
rhel01

[debian]
deb01

[linux:children]
redhat
debian

[linux]
arch01

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
- unarchive:
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

### Call a variable

A Ansible variable is called in Jinja Templating way : `{{ my_variable }}`.

You can call variable everywhere in Ansible (tasks, variables, name, ...)

### Define a variable

You can have 3 types of variables:
* String
* List
* Dict


#### String

A single Key Value where key do not have any space and value is a string (with or without quotation marks).
```
my_string: "value"
``` 

#### List

A python list representing an 'array' in other words/languages:
```yaml
my_list: ["bob", "alice"]
```

#### Dictionary

A python dictionary that structure data:
```
my_dict
    item1: value1
    item2: value2
```

### Variable precedence

You can set Ansible variables in multiple places like `group_vars`, `playbooks`, `roles` etc... but they are evaluated according to a precedence.

Here is the order of precedence from least to greatest:
```
role defaults
inventory file or script group vars
inventory group_vars/all
playbook group_vars/all
inventory group_vars/*
playbook group_vars/*
inventory file or script host vars
inventory host_vars/*
playbook host_vars/*
host facts
play vars
play vars_prompt
play vars_files
role vars (defined in role/vars/main.yml)
block vars (only for tasks in block)
task vars (only for the task)
role (and include_role) params
include params
include_vars
set_facts / registered vars
extra vars (always win precedence)
```


## Ansible Plays

A sufficient list of attributes for Ansible Play when running a playbooks:

```
- hosts: webservers
  accelerate: no
  accelerate_port: 5099
  ansible_connection: local
  any_errors_fatal: True
  become: yes
  become_method: su
  become_user: postgress
  become_flags: True
  debugger: on_failed
  gather_facts: no
  max_fail_percentage: 30
  order: sorted
  remote_user: root
  serial: 5
  strategy: debug
  vars:
    http_port: 80
  vars_files:
    - "vars.yml"
  vars_prompt:
    - name: "my_password2"
      prompt: "Enter password2"
      default: "secret"
      private: yes
      encrypt: "md5_crypt"
      confirm: yes
      salt: 1234
      salt_size: 8
  tags: 
    - stuff
  pre_tasks:
    - <task>
  roles:
    - common
    - common
      vars:
        port: 5000
      when: "bar == 'Baz'"
      tags : [one, two]
    - common
    - { role: common, port: 5000, when: "bar == 'Baz'", tags :[one, two] }
    - common
      when: month == 'Jan'
  tasks:
    - include: tasks.yaml
    - include: tasks.yaml
      vars:
        foo: aaa 
        baz:
          - z
          - y
    - { include: tasks.yaml, foo: zzz, baz: [a,b]}
    - include: tasks.yaml
      when: day == 'Thursday'
    - <task>
  post_tasks:
    - <task>
```


## Ansible Roles

### Structure of a role

Role directories strucutre:
```
roles/
└── my-role
    ├── defaults
    │   └── main.yml
    ├── files
        └── file
    ├── handlers
    │   └── main.yml
    ├── tasks
    │   └── main.yml
    ├── templates
        └── template.j2
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


