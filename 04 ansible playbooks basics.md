# 04 ansible playbooks basics
TOC:
- [04 ansible playbooks basics](#04-ansible-playbooks-basics)
  - [Using YAML for Ansible Playbooks](#using-yaml-for-ansible-playbooks)
    - [YAML code block](#yaml-code-block)
    - [Lists](#lists)
    - [Dictionaries](#dictionaries)
    - [Multiple Line Values](#multiple-line-values)
    - [When to use Quotes](#when-to-use-quotes)
  - [Creating an Ansible Play](#creating-an-ansible-play)
    - [Writing a Playbook:](#writing-a-playbook)
    - [The ansible-playbook Command](#the-ansible-playbook-command)
    - [Understanding Playbook Tasks](#understanding-playbook-tasks)
---
## Using YAML for Ansible Playbooks
---
The goal of this section is to cover YAML specifics with regard to Ansible Playbooks. 

### YAML code block
---
Best practice dictates that YAML files open with 3 hyphens, ---, and end with 3 periods, ..., as observed below.

```yaml
---
concentration: DevOps
# List of courses
courses:
- Ansible
- Openshift
- Configuration Management
- Containerized Application Development
...
```
### Lists
---
List members start with a single hyphen followed by a space. Each list item should be at the same indentation level. See the course list above.

### Dictionaries 
---
Dictionaries are key value pairs that are designated with a colon and a space.

Example:
```
course: title: Ansible level:
professional id: 123456
```

### Multiple Line Values
---
There are two characters that can be used to indicate a multi-line value: `|` or `>`

`|` will not ignore newlines in the input.

Example:
```yaml
ports: |
9001
9002
9003
```
Is interpreted as column as follows:

9001

9002

9003

`>` will ignore newlines in the input.

Example:
```yaml
Ports: >
9001
9002
9003
```
Is interpreted as one row: 9001 9002 9003

### When to use Quotes
---
- If a colon ends a line or is followed by a space, the values should be wrapped in double quotes.
- Special characters meant to be literals and should always be wrapped in double quotes.YAML Special characters are [ ] { } : > |
- It should be noted that variables in Ansible are an exception to the special character rule. As Ansible variables are indicated with curly braces, {{ variable }}, they must be wrapped in double quotes to prevent interpretation as a dictionary. Example: `port: "{{ web_port }}"`
- Booleans are automatically converted in Ansible, thus allowing one to use yes, no, true, false, etc. This means if you want something like a literal “yes” or “false”, you must use double quotes.
- Floating point numbers are taken as numbers. Sometimes you may prefer them to be a string (as in a version number). In this case, you should use double quotes.

## Creating an Ansible Play
---
What is a Play in Ansible?
- A set of instructions expressed in YAML.
- It targets a host or group of hosts.
- It provides a series of tasks (basically ad-hoc commands) that are executed in sequence.
- The goal of a play is to map a group of hosts to a well deﬁned role.
- Plays are kept in ﬁles known as Playbooks.

### Writing a Playbook:
---
- Each Playbook contains one ore more plays.
- Each play starts by designating a target which may be a host or group of hosts.
- Example:
```yaml
---
- hosts: webservers
```
- After the target is deﬁned, a number of options may be set.
  - **remote_user** — System user to execute the play (if not the current user).
  - **become** — If yes, Ansible will escalate permission to the become_user using become_method.
  - **gather_facts** — Whether or not facts should be gathered (default: yes).
- Example:
```yaml
---
- hosts: webservers
  become: yes
  remote_user: ansible
  gather_facts: yes
```
- The next section of a play is the tasks section:
  - This section contains a list of modules that will be executed against the target(s).
  - Each task maps to the use of an Ansible module.
  - Example:
```yaml
  ---
  - hosts: webservers
    become: yes
    remote_user: ansible
    gather_facts: yes
    tasks:
      - name: ensure httpd is installed
        package:
          name: httpd
          state: latest
      - name: ensure httpd is started
        service:
          name: httpd
          state: started
```
(More detail on tasks and options will be provided in future lessons!)
  
- Best Practices:
  - Comments are important!
    - Comments are indicated within the playbooks using the hash mark, `#`.
    - Break up plays with white space and lead plays with comments for easier understanding of
    a playbook.
  - Keep it Simple.
    - Try to keep plays as straightforward as possible.
    - Avoid subtly.
    - Remain consistent in feature applications.
  - Be careful with indentation!
    - Many playbook errors are the result of improper indentation.


### The ansible-playbook Command
---
**Playbooks** must be executed using the `ansible-playbook` command.

The `ansible-playbook` command takes a few basic arguments:
- The inventory ﬁle to use (use `-i` ﬂag).
- The playbook to execute.
- Example: 
```
ansible-playbook -i custom_inventory_file playbook.yml
```
This command executes the playbook **playbook.yml** using the inventory stored in the ﬁle **custom_inventory_file**.
- Notable options:
  - -K — (note capital) Asks for the become password.
  - -k — (note lowercase) Asks for the connect password.
  - -C — Run in check mode which is an effective dry run of the provided playbook.


### Understanding Playbook Tasks
---
As covered earlier, tasks are essentially the use of Ansible modules within a play.
- Tasks are presented in list form (each list element starts with a hyphen `-`) beginning with the **name** property.
- The **name** property is simply a plain English statement describing what the task does.
- The **module** to be used is provided on the next line followed by a colon.
- If applicable, each argument that is provided to the module follows line by line in the format `argument: value`.

Example:
```yaml
tasks:
- name: install elinks
  package:
    name: elinks
    state: latest
```
- Best Practices:
  - Name your tasks!
  - The provided name is displayed on task execution which provides insights to those running the plays.
  - The name also serves as basic documentation within the playbook.