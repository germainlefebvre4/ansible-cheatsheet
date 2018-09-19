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

Ansible Inventories make possible to gather servers in a single file to run commands on all these hosts in a single command.

Inventories are usually tidied by environments to let the group_vars operate on tasks and roles.

**Practice by examples**
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

A task is a YAML structure to perform and action through Ansible

Task definition :
```
- name: The task name to make things clearly
  become: yes
  become_method: sudo
  become_user: root
  check_mode: yes
  diff: no
  remote_user: ansible
  ignore_errors: True
  import_tasks: more_handlers
  include_tasks: other-tasks.yml
  notify: restart apache
  register: my_register
  changed_when: False
  failed_when: False
  vars:
    - myvar: toto
    myfiles:
      -  default.conf
  loop:
    - item1
    - ["item2", "item3"]
    - { name: "item4", description: "Desc Item 4" }
  when:
    - my_register is defined
    - ansible_distribution == "CentOS"
    - ansible_distribution_major_version == "7" 
  block:
    - name: Task to run in block
      ...
  rescue:
    - name: Task when block failed
  always:
    - name: Task always run before/after block
```


## Ansible Playbooks

Practise by examples

**Install yum package _httpd_ on RedHat servers**

```
- hosts: [redhat]
  become: true
  tasks
  - yum:
      name: httpd
      state: installed
```

**Run roles on Docker servers**
```
- hosts: [docker]
  become: true
  roles:
  - docker
  - kubernetes
```



## Ansible Variables

### Call a variable

A Ansible variable is called in Jinja Templating way : `{{ my_variable }}`.

You can call variable everywhere in Ansible (tasks, variables, name, ...)

### Define a variable

You can have 3 types of variables:
* String
* List
* Dictionary

A single Key Value where key do not have any space and value is a string (with or without quotation marks).
```
my_string: "value"
my_list: ["bob", "alice"]
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
File at `roles/example/files/my-file.sh`

Task at `roles/example/tasks/main.yml`
```
- copy:
    src: my-file.sh
    dest: /tmp/my-file.sh
```

#### Role with template to generate
Template at `roles/example/templates/my-template.sh.j2`
```
#!/bin/bash
echo "{{ timezone }}"
```

Task at `roles/example/tasks/main.yml`
```
- template:
    src: my-template.sh.j2
    dest: /tmp/my-template.sh
```

#### Role with trigger to handler
Handler at `roles/example/handlers/main.yml`
```
- name: Restart Apache
  systemd:
    name: httpd
    state: restart
```

Task at `roles/example/tasks/main.yml`
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

Task at `roles/example/tasks/main.yml`
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
  - import_tasks: roles/example/tasks/main.yml
  - include_tasks: roles/example/tasks/main.yml
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
      name: example
      tasks_from: main

```

### Include a taskfile from a role
Permit to load all variables inherant to the role.
```
- hosts: [redhat]
  tasks:
  - include_role:
      name: example
      tasks_from: install # Will call roles/example/tasks/install.yml
```

### Include tasks but filter on tag in ansible-playbook command
Only `import_role` works for including a whole role in a playbook using tags on commands.
```
- hosts: [redhat]
  tasks:
  - import_role:
      name: example
```


## Ansible Modules


