# 07 ansible roles
TOC:
- [07 ansible roles](#07-ansible-roles)
  - [What is role](#what-is-role)
  - [Roles - what is inside](#roles---what-is-inside)
  - [Roles - how to use it](#roles---how-to-use-it)
  - [Roles and variables](#roles-and-variables)

---
## What is role
---
You can think about roles like its a playbook break out into components. Each component have its own directory.

Every one can create a role and share it with others. There is roles hub (like git hub but for roles) called https://galaxy.ansible.com/ where everyone can fined needed role.

There is also an `ansible-galaxy` command that helps to deal with roles and collections with cli.

It can take a number of sub-commands:
- **role** - Perform the action on an Ansible Galaxy role. Must be combined  with  a  further  action  like  **delete/install/init** as listed below.
  - **init** - Creates  the skeleton framework of a role or collection that complies with the Galaxy metadata format. Requires a role or collection name. The collection name must be in the  format  `<namespace>.<collection>`.
  - **remove** - removes the list of roles passed as arguments from the local system.
  - **delete** - Delete a role from Ansible Galaxy.
  - **list** - List installed collections or roles.
  - **search** - searches for roles on the Ansible Galaxy server.
  - **import** - used to import a role into Ansible Galaxy.
  - **setup** - Setup an integration from Github or Travis for Ansible Galaxy roles.
  - **info** - prints  out  detailed  information  about an installed role as well as info available from the galaxy API.
  - **install** - Install one or more roles(`ansible-galaxy role  install`),  or  one  or  more  collections(`ansible-galaxy  collection install`).
- **collection** - Perform the action on an Ansible Galaxy collections. Must be combined  with  a  further  action  like  **delete/install/init**. more info : `man ansible-galaxy`

**example**:
```sh
ansible-galaxy role search apache
```

## Roles - what is inside
---

Roles provide a way of automatically loading certain var_files, tasks, and handlers based on a known file structure.
- Roles also make sharing of configuration templates easier.
- Roles require a particular directory structure.
```
base_directory/
<role_1>
    tasks/
    handlers/
    files/
    tempaltes/
    vars/
    defaults/
    meta
<role_2>
    tasks/
    defaults/
    meta/
```

- Role definitions must contain at least one of the noted directories
  - **tasks** - Contains the main list of tasks to be executed by the role.
  - **handlers** - Contains handlers, which may be used by this role or even anywhere outside this role.
  - **defaults** - Default variables for the role (see Variables for more information).
  - **vars** - Other variables for the role (see Variables for more information).
  - **files** - Contains files which can be deployed via this role.
  - **templates** - Contains templates which can be deployed via this role.
  - **meta** - Defines some meta data for this role. See below for more details.
- Unused directories need not exist.
- With the exception of **templates** and **files**, each directory **must** **include** a **main.yml** if that directory is being used.
- **main.yml** serves as the entry point for the role.
- Files in the **tasks**, **templates**, and **files** directories may be referenced without path within the role.



## Roles - how to use it
---
- To invoke a role in a playbook, you must use the `role` keyword.
- **Example** using two roles in a playbook using various keywords as needed:
```yaml
---
- hosts: webservers
  roles:
    - common
    - role: foo_app_instance
      vars:
        dir: '/opt/a'
        app_port: 5000
    - role: foo_app_instance
      vars:
        dir: '/opt/b'
        app_port: 5001
    
```
- When a role is called in a playbook, Ansible looks for the role definition in `${PWD}/roles/<role_name>`
- If `${PWD}/roles` does not contain the sought role, then `/etc/ansible/roles` is checked.
- The default role location may be changed in **ansible.cfg**.
- The full path to a role may also be specified with the role keyword to use a non-default path.
- There is also static and dynamic roles usage. 
  - **static** - `import_role` - ansible will download the role as soon as read this code line during execution even if its not applied.
  - **dynamic** - `include_role` - ansible will download the role only when this role applied to some target during execution.
- **Example** - role name vs role path and static vs dynamic:
```yaml
---
- hosts: webservers
  roles:
    - role: role_name
    - role: '/path/to/my/roles/common'
  include_role: # dynamic role usage
    - role: role_name
    - role: '/path/to/my/roles/common'
  import_role:  # static role usage
    - role: role_name
    - role: '/path/to/my/roles/common'
```
- A role with a given set of parameters will only be applied once even if called multiple times in a play.
  - If a role is called with different parameters, it will be ran again.
  - A role may have `allow_duplicates: true` defined in `meta/main.yml` within the role.
    - This will also allow the role to be applied more than once.
- Roles may have dependent roles defined in `meta/main.yml` using the `dependencies` keyword.
  - Parameters may also be included in the dependency list.
  - Dependent roles are applied prior to the role dependent on them.
  - Be careful of role duplication with dependencies.
  - **Example**:
```yaml
---
dependencies:
  - role: common
    vars:
      some_parameter: 3
  - role: apache
    vars:
      apache_port: 80
  - role: postgres
    vars:
      dbname: blarg
      other_parameter: 12
```

## Roles and variables
---
- There are three primary ways (aside from conventional variable use such as inventory) to interact with variables within a role: **vars directory**, **defaults directory**, and **parameters**.
  - Each way has a different level of precedence.
  - The **vars** directory defined within the role has the highest level of precedence. (it will override
  inventory variables as well)
  - The **defaults** directory has the lowest level of precedents and provides a ‘default’ value.
  - **Parameters** are passed inline to the role and sit between vars and defaults in terms of precedents.
- **Example** of passing a parameter
```yaml
roles:
- role: apache
  vars:
    http_port: 8080
```
- Variables defined within a role may be accessed across roles.
- You may still pass variables on the command line with the `-e` ﬂag for use in a role. (These variables
override all others in terms of precedents.)
- Best practice dictates that you properly namespace your variables when working with a role to
avoid conﬂicts.
- An easy way to do this is to prepend your role name to all variable names within the role. (Example: **webserver_timeout** instead of just **timeout**)