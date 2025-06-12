# 05 ansible playbooks essential syntax
TOC:
- [05 ansible playbooks essential syntax](#05-ansible-playbooks-essential-syntax)
  - [Define Variables](#define-variables)
  - [Using Variables in Playbooks](#using-variables-in-playbooks)
    - [Defining variables via the command line:](#defining-variables-via-the-command-line)
    - [Defining variables within a playbook with **vars**:](#defining-variables-within-a-playbook-with-vars)
    - [Defining variables within a variables ﬁle with **vars\_files**:](#defining-variables-within-a-variables-ﬁle-with-vars_files)
    - [Defining variables for host groups and/or specific host](#defining-variables-for-host-groups-andor-specific-host)
  - [Module **register**](#module-register)
  - [Working with Templates with Module **template**](#working-with-templates-with-module-template)
  - [Using Ansible Facts](#using-ansible-facts)
  - [Conditional Execution in Playbooks](#conditional-execution-in-playbooks)
  - [Using Loops in Ansible](#using-loops-in-ansible)
  - [Working with Handlers in Ansible](#working-with-handlers-in-ansible)
---

- Typical uses of variables:
  - Customize configuration values.
  - Hold return values of commands.
  - Ansible has many default variables for controlling Ansible’s behavior.


## Define Variables
---
- Variable names should be letters, numbers, and underscores.
- Variables should always start with a letter.
- Examples of valid variable names:
```yaml
foobar
foo_bar
foo5
``` 
> [!warning]
> Examples of invalid variable names:
>```yaml
>foo-bar
>1foobar
>foo.bar
>```

- Variables can be scoped by group, host, or within a playbook.
- Variables may be used to store a simple text or numeric value.
- Example: 
```yaml
month: January
```

- Variables may also be used to store simple lists.
- Example:
```yaml
colors:
- red
- blue
- yellow
```

- Additionally, variables may be used to store python style dictionaries.
- A dictionary is a list of key value pairs.
- Example:
```yaml
person:
  name: sam
  age: 4
  favorite_color: green
```



## Using Variables in Playbooks
---
- Variables may be defined in a number of ways:
  - Via command line argument.
  - Within a playbook.
  - Within a variables ﬁle.
  - Within an inventory ﬁle.

### Defining variables via the command line:
---
- Use the `--extra-vars` or `-e` ﬂag defined within a playbook.
- CLI Example: 
```sh
ansible-playbook service.yml -e "target_hosts=localhost target_service=httpd"
```

### Defining variables within a playbook with **vars**:
---
Playbook Example:
```yaml
---
- hosts: webservers
  become: yes
  vars:
    target_service: httpd
    target_state: started
  tasks:
    - name: Ensure target service is at target state
      service:
        name: "{{ target_service }}"
        state: "{{ target_state }}"
```
- **Note**: Variables are referenced using double curly braces.
- It is good practice to wrap variable names or statements containing variable names in double quotes.
- Example: `hosts: "{{ my_host_var }}"`


### Defining variables within a variables ﬁle with **vars_files**:
---
It can be any file extension while it uses yaml syntax.
Example variable ﬁle:
```yaml
# file: /home/ansible/web_vars.ini
target_service: httpd
target_state: started
```
Example Playbook:
```yaml
---
- hosts: webservers
  become: yes
  vars_files:
    /home/ansible/web_vars.ini
  tasks:
    - name: Ensure target service is at target state
      service:
        name: "{{ target_service }}"
        state: "{{ target_state }}"
```
Its possible to connect variable file in command line with `@` symbol
```sh
ansible-playbook service.yml -e @web_vars.ini
```


### Defining variables for host groups and/or specific host
---
The is an possibility to define vers for whole group.
- For host groups:
  - in home directory create an dir with name `group_vars`
  - inside that directory create ver file (or many files) with name the exactly matches the name of group inside inventory file.
  - Define variable inside files like described above.

- For hosts:
  - in home directory create an dir with name `host_vars`
  - inside that directory create ver file (or many files) with name the exactly matches the name of host inside inventory file.
  - Define variable inside files like described above.



## Module **register**
---
The register module is used to store task output in a dictionary variable.
- It essentially can save the results of a command.
- Several attributes are available: return code, stderr, and stdout.
- Example:
```yaml
- hosts: all
  tasks:
  - shell: cat /etc/motd
    register: motd_contents
    
  - shell: echo "motd contains the word hi"
    when: motd_contents.stdout.find('hi') != -1
```

## Working with Templates with Module **template**
---
Templates are files with Ansible variables inside that are substituted on play execution.

A typical use case is to deploy configuration files. And template itself is a skeleton of configuration ﬁle where variables may be used for simple customizations (such as IP addresses or host names).


- Templates use the **template** module.
- Module parameters:
  - **src** — Template ﬁle to use.
  - **dest** — Where the resulting ﬁle should be on the target host.
  - **validate** — A command that will validate a ﬁle before deployment.
  - Can also manipulate result ﬁle properties (owner, permissions, etc).
Example:
```yaml
---
- hosts: webservers
  tasks:
  - name: ensure apache is at the latest version
    yum: name=httpd state=latest
  - name: write the apache config file
    template: src=/srv/httpd.j2 dest=/etc/httpd.conf
```
- Notes regarding template files:
- They are essentially text files that have variable references.
- They use Jinja2 templating.
- They tend to be identified by using the ﬁle extension .j2.


## Using Ansible Facts
---
Ansible facts are simply various properties regarding a given remote system.
- The **setup** module can retrieve facts. The **filter** parameter takes regex to allow you to prune fact output.
- Facts are gathered by default in Ansible playbook execution. The keyword **gather_facts** may be set in playbook to change fact gathering behavior.
- Access to variable is similar to variable that you have self defined in playbook. But without defining them. Variables defined by ansible itself.
- It is possible to print Ansible facts in files using variables.
- Facts may be filtered using the setup module ad-hoc by passing a value for the filter parameter.
- It is possible to use `{{ ansible_facts }}` for conditional plays based on facts.

Example: accessing facts variables in playbook
```yaml
--- # Ansible Facts example
- hosts: localhost
  tasks:
    - name: create a file
      lineinfile:
        path: /tmp/hostname
        create: yes
        line: "{{ ansible_hostname }}"
    - name: access magic variable variant 1
      lineinfile:
        path: /tmp/hostname
        line: "{{ hostvars['localhost']['ansible_default_ipv4']['address'] }}"
    - name: access magic variable variant 2
      lineinfile:
        path: /tmp/hostname
        line: "{{ ansible_default_ipv4.gateway }}"    
        # Other magic variables are groups, group_name, and inventory_hostname
```


## Conditional Execution in Playbooks
---
Ansible playbooks are capable of making actions conditional.
- The when keyword is used to test a condition within a playbook.
- Jinja2 expressions are used for conditional evaluation.

Example using facts:
```yaml
tasks:
  - name: "shut down Debian flavored systems"
    command: /sbin/shutdown -t now
    when: ansible_os_family == "Debian"
# note that Ansible facts and vars like ansible_os_family can be used
# directly in conditionals without double curly braces
```

- It is also possible to use module output conditionally:
```yaml
- hosts: web_servers
  tasks:
  - shell: /usr/bin/foo
    register: foo_result
    ignore_errors: True
  - shell: /usr/bin/bar
    when: foo_result.rc == 5
```

- you can use special condition keywords like "exists".

Example: (If file exists do some task)
```yaml
--- # Ansible conditional example
- hosts: remote
  vars:
    target_file: /tmp/hostname
  tasks:
    - name: Gather file information
      stat:
        path: "{{ target_file }}"
      register: hostname
    - name: Rename hostname when found
      command: mv "{{ target_file }}" /tmp/net-info
      when: hostname.stat.exists 
```

## Using Loops in Ansible
---
The loop keyword may be used to more concisely express a repeated action.

Example:
```yaml
- name: add several users
user:
  name: "{{ item }}"
  state: present
  groups: "wheel"
loop:
  - testuser1
  - testuser2
```

loop may also operate with a list variable.

Example:
```yaml
- name: add several users
user:
  name: "{{ item }}"
  state: present
  groups: "wheel"
loop: "{{ user_list }}"

```
It is also possible to combine loops and conditionals:

Example:
```yaml
- name: install software on debian systems
apt:
  name: "{{ item }}"
  state: latest
loop: "{{ packages }}"
when: ansible_os_family == "Debian"
```

## Working with Handlers in Ansible
---
Ansible provides a mechanism that allows an action to be flagged for execution when a task performs
a change.
- By only executing certain tasks on change, plays are more efficient.
- This mechanism is known as a **handler** in Ansible.
- A **handler** may be called using the **notify** keyword.
- No matter how many times a **handler** is flagged in a play, it is only ran one time at the ﬁnal phase of play execution.
- **notify** will only ﬂag a **handler** if a task block makes changes.

Example:
```yaml
- name: template configuration file
  template:
    src: template.j2
    dest: /etc/foo.conf
  notify:
    - restart cache service
    - restart web services
```

- The calls made in the notify section correspond to handler definitions within the play.
- A handler may be defined similarly to tasks. Each handler section have **listen** keyword. Value must be equal to **notify** value. Listen value is notification to execute a handler.

Example:
```yaml
handlers:
- name: restart memcached
  service:
    name: memcached
    state: restarted
  listen: "restart cache service"
- name: restart apache
  service:
    name: apache
    state:restarted
  listen: "restart web services"
```

