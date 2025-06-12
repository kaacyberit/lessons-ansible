# 01 ansible install and config
TOC:
- [01 ansible install and config](#01-ansible-install-and-config)
  - [step 1 - How to Install Ansible](#step-1---how-to-install-ansible)
  - [step 2 - Conﬁgure common users (with sudo) for all hosts](#step-2---conﬁgure-common-users-with-sudo-for-all-hosts)
    - [- Creating user](#--creating-user)
    - [- Escalating permissions](#--escalating-permissions)
  - [step 3 - Conﬁgure SSH keys for Pre-shared Key Authentication](#step-3---conﬁgure-ssh-keys-for-pre-shared-key-authentication)
  - [step 4 - The Ansible Conﬁguration File](#step-4---the-ansible-conﬁguration-file)
  - [step 5 - Setting Up the Ansible inventory](#step-5---setting-up-the-ansible-inventory)
---
PREREQUESITS: it supposed that you have 3 linux machines. One control node where we will install ansible we will call it localhost. And two remote target nodes. 
For best demontration perposes one target node should be like Ubuntu distro. Another like CentOS distro.

OS installation and/or SSH installation and configuration is beyond scope of this tutorial.

## step 1 - How to Install Ansible
---
In order to install Ansible, you must conﬁgure the EPEL repository on your system.

Once the EPEL repository is conﬁgured, your package manager can install Ansible and manage de-pendencies.

For CentOS distro:

`sudo yum install ansible`

For Ubuntu distro:

`sudo apt install ansible`



## step 2 - Conﬁgure common users (with sudo) for all hosts
---
Two steps below should be performed on the Ansible control node and repeated on each other node you wish to have Ansible control.

### - Creating user
---
Ansible is best implemented using a common user across all Ansible controlled systems.

Creating a System User for Ansible

Only a basic system user with ssh access is needed for Ansible to connect to a host.

The following commands, executed as root, will create a new user on a system called `ansbile` and allow for the user password to be set:

For CentOS distro:
```bash
sudo useradd ansible
sudo passwd ansible
```
For Ubuntu distro:
```bash
sudo adduser ansible
```
### - Escalating permissions
---
For effective system management, the ansible user will need a means of privilege escalation.

`/etc/sudoers` may be edited to allow your selected user to sudo any command without password for the most automated conﬁguration using the line 

`ansible ALL=(ALL) NOPASSWD: ALL`

It is also possible to prompt for a sudo password at runtime using -K(note uppercase) if desired. (Note: This can become a challenge when executing against many systems).

## step 3 - Conﬁgure SSH keys for Pre-shared Key Authentication
---
While it is possible to connect to a remote host with Ansible using password authentication using -k (note lowercase), it is not a common practice as it can incur signiﬁcant overhead in terms of manual intervention.

More prectical way is to use passwordless or key authentication. The `ssh-keygen` and `ssh-copy-id` command can facilitate creating a pre-shared key for user authentication.

As the ansible user, ﬁrst run `ssh-keygen` without parameters and follow the prompts to create a key pair for the user. It is recommended that you perform this step on your Ansible control node.

Once the key pair has been created, run 
```bash
ssh-copy-id ansible@TARGET_HOST
``` 
to copy the ansible user’s public key to **TARGET_HOST**.

(Note: The **ansible** user must already exist on **TARGET_HOST** and be able to log in with password authentication.)

After the public key is in place, you should be able to connect to **TARGET_HOST** as the **ansible** user from your original host without being prompted for a password.



## step 4 - The Ansible Conﬁguration File
---
Ansible has a number of settings that may be adjusted. The default Ansible conﬁguration ﬁle is `/etc/ansible/ansible.cfg`

Notable conﬁgurations include:
- Default inventory conﬁguration
- Default remote user

The conﬁguration properties may be overridden using local ﬁles. 

The ﬁrst conﬁguration ﬁle found is used and later ﬁles are ignored.

Conﬁguration ﬁle which will be searched for is in the following order:
- ANSIBLE_CONFIG (environment variable if set)
- ansible.cfg (in the current directory)
- ~/.ansible.cfg (in the home directory)
- /etc/ansible/ansible.cfg

Starting at Ansible 2.4, the `ansible-config` utility ships with Ansible. The utility allows users to see:
- All the conﬁguration settings available
- Their defaults
- How to set them
- Where their current value comes from.

The utility may use a few different sub-commands:
- **view**: Displays the current conﬁg ﬁle.
- **list**: Provides all current conﬁgs reading lib/constants.py and shows env and conﬁg ﬁle setting names.
- **dump**: Shows the current settings, merges ansible.cfg if speciﬁed. 

  **Note**: the dump sub-command may take the ﬂag `--only-changed` which will only show conﬁgurations not set to the default.

- **init**: help to create initial configuration file. 

    **NOTE**: its possible that there is no cofig file after installation. In that case all setting will be default. To view defaults use `ansible-config dump`. 

To create new config file run a command:

```bash
sudo mkdir /etc/ansible
ansible-config init | sudo tee /etc/ansible/ansible.cfg > /dev/null
```
**NOTE**: ansible can thow an error that there is some mistakes in config file. in suche case you **must to comment** unnessesary options.

The `-c` or `--config` switches may be used to specify a particular conﬁguration ﬁle.

If one is not provided, the highest precedent conﬁguration ﬁle is used.

See man `ansible-config` for more information.



## step 5 - Setting Up the Ansible inventory
---
An inventory is a list of hosts that Ansible manages.

The default Ansible Inventory File is `/etc/ansible/hosts`.

Inventory location may be speciﬁed as follows:
- Default: `/etc/ansible/hosts`
- Speciﬁed by CLI: `ansible -i <custom_inventory_filename>`
- Can be set in `ansible.cfg`

Example Inventory ﬁle:
```
mail.example.com ansible_port=5556 ansible_host=192.168.0.10

[webservers]
httpd1.example.com
httpd2.example.com

[labservers]
lab[01:99]
```
- The ﬁrst line of the example ﬁle deﬁnes a host, `mail.example.com`.
- Two variables are afﬁliated with the host, `ansible_port` and `ansible_host`.
- The groups `webservers` and `labservers` are deﬁned in this example.
**Note:** the `labservers` group has **99 hosts** in it that are deﬁned via a pattern
- (The expression `lab[01:03]` is the same as specifying `lab01, lab02, lab03`.)
- Note that inventory ﬁles may be provided in YAML format but that is uncommon.
- It is recommended that you keep separate inventories for different tiers: **development**, **staging**, **production**.