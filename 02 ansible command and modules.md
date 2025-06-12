# 02 Ansible Command and Modules
TOC:
- [02 Ansible Command and Modules](#02-ansible-command-and-modules)
  - [Ansible Command](#ansible-command)
  - [Understanding Ansible Modules](#understanding-ansible-modules)
    - [Common modules (more detail on each and more modules later):](#common-modules-more-detail-on-each-and-more-modules-later)
  - [Managing Long-running Commands](#managing-long-running-commands)
  - [Parallelism in Ansible Ad-hoc](#parallelism-in-ansible-ad-hoc)
---
## Ansible Command
---
Ansible ad-hoc commands analogous to bash commands.

Playbook analogous to bash script (more on them later).

Syntax: 

```bash
ansible <HOST|HOSTGROUP> -b -m <MODULE> -a "<ARG1 ARG2 ARGN>" -f <NUM_FORKS>
```

- `HOST` is a host or host group deﬁned in the Ansible inventory ﬁle.
- `-b` is for ‘become’ (This replaces the depreciated -s ﬂag as in “sudo”). 
  - Ansible escalates permission to `--become-user` using the method deﬁned by `--become-method`.
  - Default become-user is root. (set in config file)
  - Default become-method is sudo. (set in config file)
- `-m` is for module or command to use (We will look at modules in the next section).
- `-a` is for parameters to pass. If used without module, it is like running a shell command on the target system(s).
- `-f` is used to set forks for parallelism, which is how you can have Ansible execute plays simultaneously on many hosts.

## Understanding Ansible Modules
---
Modules are discrete units of code that can be used from the command line or in a playbook task. Modules designed in such way to perfom one sprcific task. Mudules can **get** and "**understand**" the current status of target system and "**make desigion**" do module need to perfom some action to change system. 

Example: if you need to install specific version of app on all nodes. Module will automatically chech if app alreade installed. If yes - it will check version of app. After that it will decide if any changes needed and make them if needed.

And user do not need to code complex logic of checks and actions.

Modules may take arguments. Most modules take key=value arguments, delimited by whitespace.

### Common modules (more detail on each and more modules later):
---
- Module: **ping**
    - Purpose: Verify Ansible connectivity between hosts.
    - Arguments: None
- Module: **setup**
    - Purpose: Collect Ansible facts.
    - Arguments: None
- Module: **yum**
    - Purpose: Install software with yum.
    - Arguments: name=<PACKAGE_NAME> state=\<STATE\>
- Module: **service**
    - Purpose: Control service daemons.
    - Arguments: name=<SERVICE_NAME> state=\<STATE\>
- Module:**copy**
    - Purpose: Copy a ﬁle to a particular location on a target host.
    - Arguments: src=<SOURCE_PATH> dest=<ABSOLUTE_DESTINATION_PATH>

When you need help.

There are a very large number of modules that perform a variety of tasks. It is nearly impossible to know every module. That said, there is good documentation provided on **docs.ansible.com**. The module index is provided at **docs.ansible.com** and it provides detailed information on each module.

Ansible ships with the `ansible-doc` command as well. Specifying a `module_name` as a parameter will provide module speciﬁc documentation. The `-l` ﬂag will list installed modules with a brief description.

`ansible-doc -l`

`ansible-doc module_name`


## Managing Long-running Commands
---
Some operations may require a significant amount of time to execute. Ansible provides a feature to allow an operation to run asynchronously such that the status may be checked.

There are two options related to running asynchronous commands:

`-B` is used to provide a timeout value and initiate the operation.
  - Unit: seconds
  - Default value: N/A
  - Job is terminated when timeout value expires.

`-P` may be provided to engage polling of the operation on a set interval.
  - Unit: seconds
  - Default value: 15
  - Polling will leave the Ansible command in the foreground until execution on all hosts completes.
  - Set to 0 to disable polling.

Example: `ansible all -B 1200 -P 60 -a "/usr/bin/long_running_operation"`
- This example provides a 20 minute timeout and polls the operation’s status every 60 seconds.

The status may be checked using the module **async_status**.
- Example: `ansible httpd1 -m async_status -a "jid=12334568901234.1234"`
- Note: JID is only realistically obtained during playbook operation.

## Parallelism in Ansible Ad-hoc
---
Ansible uses what are know as forks to execute tasks in parallel. Each fork is able to run a task against a target host. By **default**, Ansible executes **5 forks**. Assuming one runs a module against 5 hosts, the module should execute near simultaneously on all hosts.

If more than 5 hosts are targeted, hosts after the first 5 must wait for a fork to become free before being able to execute.

The number of forks used by Ansible may be configured in ansible.cfg or on an as-needed basis using the `-f` or `--forks` ﬂag.

Note: The more forks allowed, the more system resources will be required on the Ansible control node.

Example: `ansible all -f 10 -m setup`
- This example will use 10 forks to execute the setup module against all hosts in the inventory.
- Note: Ansible will only fork as much as required.
- Presuming Ansible is configured to run 10 forks but is only tasked with executing on 6 hosts, only 6 forks will be created.