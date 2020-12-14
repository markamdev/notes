# Ansible learning course notes

This document is a notebook for Ansible courses (basic and andvanced) by Mumshad Mannanbeth on Udemy:

* [Ansible for Absolute Beginner](https://udemy.com/course/learn-ansible/learn/)
* [Ansible Advanced](https://udemy.com/course/learn-ansible-advanced/learn/)

## Inventory file

* By default stored in /etc/ansible/inventory
* Can be composed of:
* * List of IP addresses
* * List of FQDN
* * Aliases in form

```conf
somename ansible_host=server.domain.com
```

* * groups preceded by `[groupname]`
* Possible inventory params:
* * ansible\_host
* * ansible\_connection (ssh, winrm or localhost)
* * ansible\_port
* * ansible\_user
* * ansible\_ssh\_pass or ansible\_password
* * group of groups: `[groupname:children]`

## YAML quick start

* * key-value pairs

```yaml
key1: value1
key2: value2
key3: value3
key4: value4
```

* * lists
```yaml
List\_name:
- item1
- item1

List\_2\_name:
- item-1-list-2
- item-2-list-2
```

* * dictionaries

```yaml
dictionary-name:
    key1:value1
    key2:value2
    kayX:valueY

dictionary2-name:
    keyx: valuen
    keyy: valueo
    keyz: valuep
```

## Playbooks

* Ansible **Playbook** is a single YAML file
* * **Play** defines set of tasks to be run on hosts
* * * **Task** is an action to be performed (execute command, run script)

Example:

```yaml
-
  name: play-name
  hosts: localhost
  tasks:
   - name: Execute command date
     command: date

   - name: Execute some scrpipt
     script: some_script.sh

   - name: install http service
     yum:
       name: httpd
       state: present

   - name: start web server
     service:
       name: httpd
       state: started
```

## Variables

* Can be defined in *inventory* file for each host

```yaml
hostname var1=x var2=y var3=z
```

* Can be defined in Playbook before tasks:

```yaml
vars:
    some_var: x.y.z
tasks:
    -
        name: "Some task"
        module:
            param: 'text {{some_var}}'
```

* Can be defined in separate file *variables*:

```yaml
variable1: value1
variableX: valuex
```

## Conditionals

Used to execute task when condition is met:

```yaml
-
    name: 'Some play'
    hosts: all_hosts
    tasks:
    -
        name: "Task1"
        command: some-command
        when: variable == 'value'
    -
        name: "Task2"
        command: other-command
        when: variable == 'value2'
```

## Loops

Executing same task multiple times with modified parameter

* *loop* directive

```yaml
-
    name: 'Do something'
    hosts: all
    tasks:
    -
        name: 'Do something with xval'
        module: xval= {{item}}
        loop:
        - val-1
        - val-2
        - val-3
```

* *with\_\** directive
* * *with\_items*
* * *with\_url*
* * *with\_file*
* * *with\_mongodb*
* * *with\_\<lookupplugin\>*


## Roles (intro)

Pre-defined *playbooks* with all elements orgnized in separate files/directories for: *tasks*, *variables*, *defaults*, *handlers* and *templates*.

* *ansible-galaxy* is a tool for *roles* usage
* create new role using `ansible-galaxy init rolename`
* search community for roles using `ansible-galaxy search keyword`
* download role from community using `ansible-galaxy install rolename`

## Variables separation

Host variables can be stored in separate file for each host from inventory file.

Original inventory file:

```txt
target1 ansible_host=192.168.163.114 ansible_ssh_pass=osboxes.org
target2 ansible_host=192.168.163.115 ansible_ssh_pass=osboxes.org
```

can be changed into:

```txt
target1
target2
```

with two additional files (*target1.yml* and *target2.yml* respectively) stored inside *host_vars* folder. File content for *target1.yml* should be as follow:

```yaml
ansible_host: 192.168.163.114
ansible_ssh_pass: osboxes.org
```

In the same way variable can be defined for hosts grouped in *inventory* file. The variable file *groupname.yaml* has to be placed in *group_vars* folder.


## Include keyword

It is possible to create files with task(s) and then include them in Play using syntax below:

```yaml
- name: Some play
  hosts: host_list
  tasks:
  - include: tasks/file1.yml
  - include: tasks/file2.yml
```

## Roles (usage)

Roles have pre-defined, unified set of folders. It is like list of tasks but with description, variables and metadata. Roles can be shared in community using [Ansible Galaxy](https://galaxy.ansible.com).

To use role (stored localy) in play(book):

```yaml
- name: Some play with roles
  hosts: host_list
  roles:
  - rolename1
  - rolename2
```

## Async tasks

Tasks can execute long time and/or give a result after some time. To avoid keeping long SSH session by Ansible it is possible to launch a command and check its' status later:

```yaml
- name: Some play with async task
  hosts: host_list
  tasks:
  - name: Some async tasks
    command: /opt/do_something.sh
    async: 360 # timeout [s]
    poll: 60 # re-check time in [s]
```

If `poll` is set to `0` then Ansible is not waiting for task result but immediately executes next one. To be able to later "go back" and check task result the `register` has to be used

```yaml
- name: Some play with async task
  hosts: host_list
  tasks:
  - name: Some async tasks
    command: /opt/do_something.sh
    async: 360 # timeout [s]
    poll: 0 # do not poll periodically
    register: task1_result

  - name: Some async tasks 2
    command: /opt/do_something_else.sh
    async: 240 # timeout [s]
    poll: 0 # do not poll periodically
    register: task2_result

  - name: "Status checker"
    async_status: jid={{ task1_result.ansible_job_id }}
    register: job_result
    until: job_result.finished
    retries: 30
```

## Strategy

Defines how Ansible has to execute multiple tasks on multiple servers. Available strategies are:

* **linear** (default)
* * tasks are executed one after another
* * each task is executed on each server
* * next task is taken when current one succeed on all server
* * selected by `strategy: linear` or by default when no strategy defined
* **free**
* * next task on current server is taken when current one succeed, without waiting for other servers
* * selected by `strategy: free`
* **batch**
* * works as **linear** but with limitation of hosts used at once
* * selected by `serial: X` where *X* is number of hosts or `serial: "XY%"` to select number of hosts proportionally
* * it possible to define array of number of hosts to be used in sequence (ex. first execute on 3 hosts, if succedd try next 4, if succeed try ...)

## Ansible forks

Ansible is executing tasks on each server using separate *fork*. Number of forks limits the maximum number of hosts configured at once. This value can be configured in *ansible.cfg* in `forks = X` option. Default setting is *5*.

## Error handling

By default Ansible exits playbook execution on first task failed. In a single-server scenario it's reasonable. When multiple servers used then Ansible is breaking playbook execution on this particular server but continues on the others.

Default behavior can be changed to *break all when any fails* using playbook option `any_errors_fatal: true`.

Task failure can be ignored by `ignore_errors: true`. Task failure can be defined using `failed_when: ...` option like in example below:

```yaml
- name: Some play
  hosts: host_list
  task:
  - ...

  - ...

  - name: Execute something
    command: do_something
    register: ds_output
    failed_when: "'Error' in ds_output.stdout"
```

## Jinja2 templating

To replace static content with some dynamically generated/substituted values templating can be used. Variable (ex. inside a text string) is marked by *{{* and *}}*.

There are multiple option (filters) possisble for variables in template:

* {{ var_name | default("unknown") }} - set default value for variable *var_name* if none given
* {{ var_name | upper }} - set string variable to uppercase
* {{ var_name | lower }} - set string variable to lowercase
* {{ var_name | title }} - set string variable to "titlecase" (only first letter in capital)
* {{ var_name | replace("something","nothing") }} - replace word in string variable
* {{ var_name | min }} - minimal value from given list (array) of numbers
* {{ var_name | max }} - maximal value from given list (array) of numbers
* {{ var_name | unique }} - remove duplication from given list (array) of numbers
* {{ var_name | union([x, y, z]) }} - append values [x, y, z] to the  given list
* {{ X |  random}} - generate random value from 1 to X
* {{ "A", "B", "C" | join(":")}} - merge given array of strings using ":" as a separator

Some more complex filters for data formatting and validation can be found in ansible [documentation](https://docs.ansible.com/ansible/latest/user_guide/playbooks_filters.html#filters-for-formatting-data).

## Lookup

Plugin for reading some data (variables) for different file formats including CSV, INI or databases.

## Vault

Plugin for encrypting sensible data (ex. password for host access)

## Dynamic Inventory

Replacing static *inventory.txt* file with data fetched from some service (command, cloud API and so). Possible option is *Python* or *bash* script.

Script should handle following options/arguments:
* *--list* - provide full inventory file
* hostname(s) - provide data for specific host(s)

## Custom modules

Ansible can be extended with custom, user defined modules written as a Python scripts. For details see ansible documentation.
