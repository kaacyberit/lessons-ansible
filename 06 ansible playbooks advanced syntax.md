# 06 ansible playbooks advanced syntax
TOC:
- [06 ansible playbooks advanced syntax](#06-ansible-playbooks-advanced-syntax)
  - [Advanced inventory configuration](#advanced-inventory-configuration)
  - [Executing Selective Parts of a Playbooks](#executing-selective-parts-of-a-playbooks)
  - [Working with Sensitive Data using Ansible Vault](#working-with-sensitive-data-using-ansible-vault)
  - [Error Handling in a Playbook](#error-handling-in-a-playbook)
  - [Asynchronous Tasks within a Playbook](#asynchronous-tasks-within-a-playbook)
  - [Delegating Playbook Execution](#delegating-playbook-execution)
  - [Parallelism in playbooks](#parallelism-in-playbooks)
  - [Using run\_once](#using-run_once)

## Advanced inventory configuration
---
An inventory is a list of hosts that Ansible manages.

Inventory can be **STATIC** or **DYNAMIC**

The default Ansible Inventory **STATIC** File. Default is /etc/ansible/hosts.

Ansible Inventory can by **DYNAMIC**. It means that list of host provided by script. For example shell script or python script.

Main contributors of dynamic inventory scripts is cloud providers. 

Dynamic Inventory script may be specified as follows:

```
- Specified by CLI: ansible -i <custom_inventory_filename.sh>
- Can be set in ansible.cfg
```


## Executing Selective Parts of a Playbooks
---
- Ansible allows for both plays and tasks to be tagged.
- By tagging a play or task, you may run a playbooks in such a way as to only run plays or tasks with a particular tag.
- Alternatively, you may also skip certain tags during execution.
- **Note**: Tasks can be tagged the same.

Example:
```yaml
tasks:
- name: install software
  yum:
    name: "{{ item }}"
    state: installed
  loop:
    - httpd
    - memcached
  tags:
    - packages

- name: install conf file
  template:
    src: templates/src.j2
    dest: /etc/foo.conf
  tags:
    - configuration
```

- You specify which tags to run or not run via arguments to the **ansible-playbook **command.
- Run certain tag CLI syntax: 
```sh
ansible-playbook all playbook.yml --tags "pacakges"
```
  - Skip tag CLI syntax:
```sh
 ansible-playbook all playbook.yml --skip-tags "configuration"
```

## Working with Sensitive Data using Ansible Vault
---
The **ansible-vault** command is used to encrypt files and work with those files.
- It can take a number of sub-commands:
  - **encrypt** - to protect a ﬁle: `ansible-vault encrypt <file>`
  - **rekey** - to change the password of an already encrypted ﬁle: `ansible-vault rekey <file>`
  - **view** - to output the contents of an encrypted ﬁle: `ansible-vault view <file>`
  - **edit** - to edit an encrypted ﬁle: `ansible-vault edit <file>`
  - **decrypt** - will unencrypt an encrypted ﬁle: `ansible-vault decrypt <file>`
  - **encrypt_string** - will encrypt a string: `ansible-vault encrypt_string 'encrypted text goes here'`
- An ﬁle encrypted with **ansible-vault** is called a **vault** in Ansible parlance.
- The primary use case for **vaults** is for encrypting variable files to protect sensitive information such as passwords.
- It is also possible to encrypt task files or even arbitrary files such as binaries if desired.
- The **ansible-vault** password ﬁle is simply a ﬁle that contains a password. There is no special formatting.
- The recommended way to provide a vault password from the CLI is to use `--vault-id`.
  - `--vault-id` may be passed a vault ﬁle or a prompt ﬂag (–vault-id@prompt) to collect credentials to unencrypt a target vault.
  - Prior to –vault-id, Ansible could only take a single vault password for a playbook.
  - Multiple Vault IDs may be provided and Ansible will try each sequentially to unencrypt as needed.
  - Vault IDs also allow for the application of labels to encrypted strings.
  - **Example**:
      ```sh
      ansible-vault encrypt_string --vault-id test@my-vault-file 'some secret text' > file.txt
      ```
    - The label ‘**test**’ is applied using the password from the vault ﬁle **my-vault-file.**
    - In order for Ansible to use the vault-id during playbook execution you must pass `--vault-id`
    `test@my-vault-file` with the `ansible-playbook` command.
  - **Example**:
    - Let us say you have a playbook called site.yml that makes use of the vault file.txt.
    - In order for Ansible to access the file.txt vault, you must specify the password ﬁle for the
    vault using the correct vault id.
    - Run: 
      ```sh
      ansible-playbook --vault-id test@my-vault-file site.yml
      ```
    - Using this command, Ansible will try the password from my-vault-file on any string labeled
    with ‘test’ before trying any other passwords or vault files.
  - You may also specify **@prompt** instead of **label@password_file** to have Ansible prompt for the password.
  - Labels are not strictly required.
    - You may use only a password ﬁle.
    - Generally, this is not ideal but may have niche use cases.
    - **Note**: Password files may also be executable (such as a python script).
  - **Note**: when debugging plays, it is possible that sensitive information may be displayed in verbose logs.
  - You can set `no_log` for a module to censor log output to avoid accidentally exposing sensitive
  information during play execution.
```yaml
--- # Vault example
- hosts: localhost
  vars_files:
    /home/ansible/vault
  tasks:
    - name: Add secret text to open.txt
      lineinfile:
        path: /home/ansible/open.txt
        create: yes
        line: "{{ password }}"
      no_log: true
```



## Error Handling in a Playbook
---
- A playbook may be ran against specific hosts and groups out of what is designated within the playbook with `--limit` flag.
- **Example**: `ansible-playbook <playbook> --limit <hostname>`
- May alternatively specify a list ﬁle using `--limit @filename`
- May also use after playbook failure.
  - When a playbook fails to execute on any host, a ﬁle is created containing the names of each
  host where the playbook failed.
  - This ﬁle may be used with the `--limit @filename` ﬂag to execute only against hosts where the playbook
  failed.
- Ansible may be configured to continue execution even after an error occurs.
- **Example**: in playbook write `ignore_errors: yes`
- When set for a task, playbooks will not halt on that task failing.

Ansible allows failure conditions to be defined with `failed_when` keyword.
- Allows you to specify the failure condition for a given task.
- **Example**:
```yaml
  - name: Fail task when both files are identical
    raw: diff foo/file1 bar/file2
    register: diff_cmd
    failed_when: diff_cmd.rc == 0 or diff_cmd.rc >= 2
```

There is also **changed_when** keyword.
- This keyword allows overriding what Ansible considers changed.
- Using a Jinja2 expression on output to create the rule.
- **Example**:
```yaml
- name: Run foo process
  shell: /usr/local/foo
  register: foo_result
  changed_when: "foo_result.rc != 2"
```

The **debug** module may be used to help troubleshoot plays.
- Use to print detail information about in progress plays.
- Handy for troubleshooting.
- Debug takes two primary parameters that are mutually exclusive:
  - **msg** - A message that is printed to STDOUT
  - **var** - A variable whose content is printed to STDOUT.
- **Example**:
```yaml
- debug:
  msg: "System {{ inventory_hostname }} has uuid {{ ansible_product_uuid }}"
```

Error handling may also be dealt with using **block** groups in Ansible.
- There are 3 key blocks that may be used to organize tasks:
  - **block** - Group tasks into a ‘block’.
  - **rescue** - A special block that is executed when the preceding **block** fails.
  - **always** - A special block that is always executed after the preceding **block**.
- **Example**:
```yaml
tasks:
  - name: Attempt and gracefully roll back demo
    block:
      - debug:
        msg: 'I execute normally'
      - command: /bin/false
      - debug:
        msg: 'I never execute, due to the above task failing'
    rescue:
      - debug:
        msg: 'I caught an error'
      - command: /bin/false
      - debug:
        msg: 'I also never execute :-('
    always:
      - debug:
        msg: "this always executes"
```


## Asynchronous Tasks within a Playbook
---
Some operations may require a significant amount of time to execute.
- By default, all playbook block tasks ran against a single host use a single SSH session.
- Ansible provides the async feature to allow an operation to run asynchronously such that the status may be checked.
- This can prevent interruption from SSH timeouts for long running operations.
- You may configure a few key values for an asynchronous task:
  - A timeout for an operation (default is unlimited).
  - A poll value for who often Ansible should check back.
  - Default poll value is 10 seconds.
  - **Note**: A poll value of 0 will have Ansible not check back on a task.
- **Example**:
```yaml
- name: 'Install docker-io (async)'
  yum:
    name: docker-io
    state: installed
  async: 1000
  poll: 25
```

## Delegating Playbook Execution
---
Certain tasks may need to be executed on specific hosts.
- This is referred to as delegation in Ansible.
- By delegating a task, the task will only run on the host or group to which it was delegated.
- In order to delegate, use the `delegate_to` keyword.
- **Example**:
```yaml
- hosts: webservers
  tasks:
    - name: take out of load balancer pool
      command: /usr/bin/take_out_of_pool {{ inventory_hostname }}
      delegate_to: 127.0.0.1

    - name: Update software package
      yum:
        name: acme-web-stack
        state: latest
```
- In the example, the task, ‘take out of load balancer pool’, would be ran specifically on 127.0.0.1 an no other host in webservers.
- You may also use DNS names over IP addresses if preferred.
- Delegating to localhost may also be expressed in shorthand using `local_action: <module_name>
[arg1=val1] ... [argN=valN]` for a given task.


## Parallelism in playbooks
---
It is possible to control number of host acted upon at once time by Ansible.
- This may be done using **forks**, which are parallel Ansible processes that execute playbook tasks.
- The number of forks can be set using `-f` ﬂag with the `ansible` or `ansible-playbook` commands.
- The default number of forks is 5 but can be set in ansible.cfg.
 
The `serial` keyword may be used to control forks in playbook.
- You may provide as integer count or as percentage.
- You may provide a step **up approach** (can mix and match count with percentage)
- It is possible to use `max_fail_percentage` to allow a certain percentage to fail (Ansible will still pass play).
- **Note**: serial can only be as parallel as the number of set forks will allow.
- **Example**:
```yaml
---
- hosts: all
  max_fail_percentage: 10 # if more then 10% fails - stop
  serial:
    - 1
    - 5
    - "30%"
  tasks:
    - name: Install apache
      yum: name=httpd state=latest
```


## Using run_once
---
- There are scenarios where a specific task needs to be ran only a single time in a given playbook and not on each host.
- This may be achieved using the run_once keyword.
- **Example**:
```yaml
- name: db upgrade task
  command: /opt/application/upgrade_db.py
  run_once: true
```
- This may be leveraged with `delegate_to` for greater control over which host executes the command.
- It should also be noted that when used with serial that run_once will execute for each serial batch.