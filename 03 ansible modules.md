# 03 ansible modules
TOC:
- [03 ansible modules](#03-ansible-modules)
  - [Modules Command, Shell and Script](#modules-command-shell-and-script)
    - [module **command**](#module-command)
    - [module **shell**](#module-shell)
    - [module **script**](#module-script)
  - [Modules for host check](#modules-for-host-check)
    - [module **ping**](#module-ping)
  - [Modules for Collecting System information](#modules-for-collecting-system-information)
    - [module **setup** - retrieve facts.](#module-setup---retrieve-facts)
  - [Modules for Working with files. Modules File and Copy](#modules-for-working-with-files-modules-file-and-copy)
    - [module **file**](#module-file)
    - [module **copy**](#module-copy)
  - [Modules for working with files - Downloading Files](#modules-for-working-with-files---downloading-files)
    - [Module **get\_url**](#module-get_url)
  - [Modules for working with files - Working with File Archives](#modules-for-working-with-files---working-with-file-archives)
    - [Modules **archive** and **unarchive**](#modules-archive-and-unarchive)
  - [Modules for working with files - Editing File Contents](#modules-for-working-with-files---editing-file-contents)
    - [module **lineinfile**](#module-lineinfile)
    - [module **replace**](#module-replace)
  - [Modules for manage System Users an Groups](#modules-for-manage-system-users-an-groups)
    - [module **user**](#module-user)
    - [module **group**](#module-group)
  - [Modules for manage software/APPs](#modules-for-manage-softwareapps)
    - [modules **package**, **yum**, and **apt**](#modules-package-yum-and-apt)
  - [Modules for manage Daemons](#modules-for-manage-daemons)
    - [modules **service**](#modules-service)
---
## Modules Command, Shell and Script
----
The **shell** and **command** modules are both effective ways to run ‘raw’ commands on a target host.

### module **command**
---
It is the **default module** used with Ansible ad-hoc when no module is specified.

The arguments provided are ‘free form’ and ran as a command on each specified remote host.

The module itself may take a few parameters/arguments:
- **creates**: Execution of the module results in the creation of the provided ﬁle.
- **removes**: Execution of the module results in the removal of the provided ﬁle.
- **chdir**: Change to the provided directory before executing the command.

### module **shell**
---
Works almost exactly like the command module except commands are ran in a shell environment (using /bin/sh).

The primary reason the shell module might be used over the command module is for use of **input / output redirection, environment variables,** or other such shell features (which are
not supported with the command module).

Per the Ansible documentation, the best practice is to use the command module unless you require shell features.

### module **script**
---
If running multiple shell commands is necessary, there is a **script** module that will copy a shell script to a remote host and execute it, featuring the same arguments that are supported by the shell command. (Alternatively, you can create your own Ansible module!)



## Modules for host check
---
### module **ping**
---
Verify Ansible connectivity between hosts.

The module itself may take a few parameters/arguments:
- **data**: (string) value to return as response. If not specified default value is "pong".

`ansible localhost -m ping -a "data=hello"`

If **data=crash** response will throw an exception with more info, that some times can be useful.

`ansible localhost -m ping -a "data=crash"`



## Modules for Collecting System information
---
Ansible facts are simply various properties regarding a given remote system.
### module **setup** - retrieve facts.
---
The **filter** parameter takes regex to allow you to prune/filter fact output.

`ansible localhost -m setup -a 'filter=ansible_*_mb'`

Facts are gathered by default in Ansible playbook execution. The keyword **gather_facts** may be set in a playbook to change fact gathering behavior. It is possible to print Ansible facts in files using variables.

Ansible command output may be directed to a ﬁle using the `--tree outputfile` ﬂag, which may be helpful when working with facts.

It is possible to use `{{ ansible_facts }}` for conditional plays based on facts.




## Modules for Working with files. Modules File and Copy 
---
Working with files is an essential operation for systems management and configuration. Ansible has a number of core modules that provide for ﬁle manipulation that will be covered throughout this course.

### module **file** 
---
The file module can be used to create, delete, and modify ﬁle properties.

`ansible localhost -m file -a 'path=/tmp/foo.txt state=absent'`

The following are notable arguments used by the copy module: ("=" - mandatory option)
- = **path**: Path to the file being managed. (aliases: [dest, name])
- **state**: choices: [absent, directory, file, hard, link, touch]



### module **copy** 
---
This module is used for copying files.
- Files may be copied from a number of sources.
- Files may be copied from the control node to target nodes.
- Files may be copied from ﬁle systems on the target node.
- Files may be created during execution of the copy module and place on target nodes.

The following are notable arguments used by the copy module: ("=" - mandatory option)
- **remote_src**
  - If no (default option), it will search for src on the control node.
  - If yes it will go to the target node for the src.
  - Currently remote_src does not support recursive copying.
  - **remote_src** only works with mode=preserve as of version 2.6.
- **content**: When used instead of src, sets the contents of a ﬁle directly to the specified value.
- = **dest**: Remote absolute path where the ﬁle should be copied to.
- **mode** - Mode (in octal or symbolic notation) the ﬁle or directory should be.
- **src** - Local path to a ﬁle to copy to the remote server; can be absolute or relative.
  - If path is a directory, it is copied recursively.
    - In this case, if path ends with “/”, only contents inside of that directory are copied to destination.
    - Otherwise, if it does not end with “/”, the directory itself with all contents is copied.
    - This behavior is similar to rsync.


## Modules for working with files - Downloading Files
---
### Module **get_url**
---
It is frequently necessary to download files from HTTP hosts on managed hosts. The **get_url** module provides functionality for downloaded files over HTTP with a variety of options.
- The HTTP, HTTPS, and FTP protocols are supported
- Basic and SSL Authentication
- Checksum verification
- Proxy network access

Notable Arguments: ("=" - mandatory option)
- **dest**: Where the ﬁle should be placed on the remote host.
- **checksum** - The algorithm and result value used to verify the ﬁle’s checksum.
  - **Note**: The result value may be provided as a text ﬁle containing the value or the plan text value.
- **force_basic_auth**: This option forces the sending of the Basic authentication header upon initial request.
- **mode**: The mode of the final ﬁle.
- **url**: The URL of the ﬁle to download.


## Modules for working with files - Working with File Archives
---
### Modules **archive** and **unarchive**
---
The archive and unarchive modules may be used to to work with various types of ﬁle archives. 

These modules work with common archives such as **tar**, **gunzip** (default), and **zip**.

These modules allow for compression (or expansion) of files and directories.

**archive** Notable Arguments: ("=" - mandatory option)
- **dest**: The ﬁle name of the archive ﬁle.
- **format**: Select compression format.
- = **path**: Remote absolute path, glob, or list of paths or globs for the files to archive.
- **remove**: Controls if the source files will be deleted after the archive is created.

**unarchive** notable arguments:  ("=" - mandatory option)
- = **dest**: Where archive files should be unpacked.
- **remote_src**: If set to ‘yes’, ﬁle is already on target system.
- **remove**: Controls if the source archive ﬁle will be deleted after execution.
- **src**: Location of source archive.
  - If **remote_src** is set to ‘no’ (the default) **src** is local to control server.
  - If **remote_src** is set to ‘yes’, **src** is on target server.
  - If **remote_src** is set to ‘yes’ and **src** contains ://, the target server will download the ﬁle from specified URL (similar to get_url module).



## Modules for working with files - Editing File Contents
---
### module **lineinfile**
---
Key way of interacting with a ﬁle’s contents.

The key feature of the module is inserting a line of provided text into a given ﬁle.
- The module will place the line of text at the end of a ﬁle by default, however it can place the line after
or before a line of matched text.
- If the module detects the provided text already in place, it will not add it again.
- It may also remove lines of text.

Key arguments for the lineinfile module: ("=" - mandatory option)
- **backup**: Create a backup ﬁle, including the timestamp information, so you can get the original ﬁle
back if you somehow clobbered it incorrectly.
- **create**: Create the ﬁle if it does not exist. (False by default.)
- **state**: Whether or not the line should be there.
- **insertafter**: Takes a regular expression and inserts line after the first matched line. (Only honored if regexp is undeﬁned or unmatched.)
- **insertbefore**: Takes a regular expression and inserts line before the first matched line. (Only honored if regexp is undefined or unmatched.)
- **regexp**: Takes a regular expression and replaces the last line matched with line or removes the matched line if state is set to absent.
- **line**: The text to insert/replace.
- = **path**: The ﬁle to modify.

**NOTE**: by default module do not create file if it does not exist. If you want to create file use `create=yes` in ad-hoc commands and `create: yes` in playbook.


### module **replace**  
---
Can make more granular ﬁle changes using regular expressions.
- It uses a replace argument instead of the line argument.
- The module also supports before, after, path, and regexp arguments.


## Modules for manage System Users an Groups
---
### module **user**
---
Interacting with system users is a common operational task. Ansible allows this functionality using the **user** module. The user module may interact with users in many standard ways.
- Create users
- Remove users
- Change user properties

Common Arguments: ("=" - mandatory option)
- **group**: Specify a primary group for the named user.
- **groups**: Specify secondary groups for the named user.
- = **name**: Name of the user with which to interact.
- **generate_ssh_key**: Whether to generate a SSH key for the named user.
- **remove**: When used with state=absent, the named user is deleted.
- **state**: Creates the named user if it doesn’t exist when set to ‘present’.
- **uid**: Optionally set the UID of the named user.

`ansible localhost -m user -a 'name=root` - check if root user exists

`ansible localhost -b -K -m user -a 'name=ansible state=present` - check if ansible user exists if no - create it.

**NOTE**: manipulation with users need root privileges.There for command need -b option

`-b` - become a root user.

`-K` - (upper case K) - ask sudo password.

### module **group**
---
Just as with system users, Ansible support working with system groups. The **group** module serves this function. The **group** module works with groups in the same way the user module interacts with users.

Common Arguments: ("=" - mandatory option)
- **gid**: Optionally set the GID of the named group.
- =**name**: Name of the group with which to interact.
- **state**: Creates the named group if it doesn’t exist when set to ‘present’.
- **system**: Indicate if a group is a system group.



## Modules for manage software/APPs
---
### modules **package**, **yum**, and **apt**
---
One of the key features necessary for bootstrapping and managing a host is the ability to install and manage software packages.

Modules **package**, **yum**, **and** apt are used for this purpose depending on distribution.

**yum** and **apt** are used for distribution specific software management.

The **package** module will automatically detect the target host distribution and use the appropriate method of installing software.

In a mixed distribution environment, the package module is best practice.

Common Arguments: ("=" - mandatory option)
- = **name**: The name of the software package to be installed.
- **state**: If the named package should be ‘present’ or ‘absent’.
- **user**: Which package manage to use. (Defaults to ‘auto’.)

`ansible localhost -m package -a 'name=mc state=latest'`

**NOTE**: package names could be different in different distros. Like web server name: **apache2** for **apt** and **httpd** for **yum**. In such cases there is always will be an error. TO FIX this you can create inventory groups based on distro. And run specific command for each distro. Another way is to use conditions. More on them later.



## Modules for manage Daemons
---
### modules **service**
---
Another common administration task is to control system daemons. The **service** module serves this purpose. It is compatible with both **systemd** and the **older BSD init** style of daemon management.

- Common Arguments: ("=" - mandatory option)
- **enabled**: Configures whether or not the service should start on boot.
- = **name**: The name of the service to manage.
- **state**: The state of the service after the module executes. May be one of:
  - **started**: Service will be started (will not restart if already started).
  - **stopped**: Service will be stopped.
  - **restarted**: Service will be restarted even if it is already started.
  - **reloaded**: Service configuration will be reloaded if supported by the service.