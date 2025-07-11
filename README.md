# Automated Virtual Machine Lifecycle

Welcome to the _Automated Virtual Machine Lifecycle_ project!

- [Motivation](#motivation)
- [Purpose](#purpose)
- [Supported versions](#supported-versions)
- [Why Ansible?](#why-ansible)
- [Directory Structure](#directory-structure)
- [Main Configuration File](#main-configuration-file)
- [How to run this?](#how-to-run-this)

<br />

## Motivation
Most of the time virtual machines provisioning does not come alone, I mean, it isn't just about _creating_, _spinning off_ or _cloning_ a [golden image](https://www.techtarget.com/searchitoperations/definition/golden-image).

In situations where we need to do extra steps to really say "I've provisioned a virtual machine", I've developed this **Ansible** project to take care of a bunch of additional configuration.

> [!NOTE]
> [VMWare Sphere](https://www.vmware.com/products/cloud-infrastructure/vsphere) is the virtualization platform to run the virtual machines that will be created and terminated by these scripts.

<br />

## Purpose
Specifically speaking, this **Ansible** project takes care of:

* Reserving a static IP address and setting up a DNS record for the virtual machine on _Men & Mice_ (now [Micetro](https://bluecatnetworks.com/products/micetro/ipam/))
* _Optionally_, create an extra `custom` DNS record on _Men & Mice_ (now [Micetro](https://bluecatnetworks.com/products/micetro/ipam/))
* Cloning an existing VMWare _Template_ (either Linux or Microsoft Windows)
* _Optionally_, add extra virtual disk(s) from specified datastores.
* _Optionally_, add extra virtual network interface(s) bound to different network(s).
* Powering on the newly-created virtual machine.
* Terminating a virtual machine, and deleting any associated resources on _Men & Mice_ and _Active Directory_.

These Ansible playbooks provide a thorough virtual machine lifecycle (through provisioning and termination steps) which is great for ephemeral environments.

<br />

## Supported versions
This project has been tested on the following versions:

| Component    | Version |
| :----------: | :---------: |
| VMWare vSphere | 5.x, 6.x |
| Ansible  | 2.x  | 2.x    |
| Men & Mice   | 9.x   |

<br />

## Why Ansible?
Despite of other automation tools out there that could also get the job done, _Ansible_ does have certain pros such as:

- Implement advanced logic on your automation.
- Easily understandable Python language.
- Playbooks are written in YAML.
- Lots of community-managed modules to choose from.
 
<br />

## Directory Structure

This project has the following directory structure:

```
(root)
+-- provisioning/                                 # Directory where all the provisioning playbooks are in
|   +-- provisioning_dns_management.yml           # DNS setup playbook
|   +-- provisioning_extra_dns_management.yml     # DNS setup playbook for extra records
|   +-- provisioning_vm_management.yml            # Virtual Machine setup playbook
|   +-- provisioning_extra_disk_management.yml    # Extra Virtual Disk setup playbook
|
+-- termination/                                  # Directory where all the termination playbooks are in
|   +-- termination_dns_management.yml            # DNS termination playbook
|   +-- termination_ldap_management.yml           # LDAP termination playbook
|   +-- termination_vm_management.yml             # Virtual Machine termination playbook
|
+-- roles/                                        # Ansible roles
|   +-- ldap/                                     # `LDAP` role
|   +-- mmice/                                    # `Men & Mice` (IPAM) role
|   +-- vmware/                                   # `VMWare vSphere` role
|
+-- runtime/                                      # Directory where utility playbooks are in
|   +-- runtime_checks.yml                        # This playbook runs a series of checks before any provisioning or termination 
|   +-- runtime_facts.yml                         # This playbook sets a series to facts for the provisioning or termination to work
|
+-- vars/                                         # Directory where the global settings (variables) are in
|   +-- environment.yml                           # Main configuration file where custom environment settings are placed
|
+-- ansible.cfg                                   # Ansible configuration file
+-- requirements.txt                              # Python requirements file
+-- provision_vm.yml                              # Playbook to kick off a Virtual Machine provisioning
+-- terminate_vm.yml                              # Playbook to kick off a Virtual Machine termination
```

<br />

## Main Configuration File
This project's main configuration file is `vars/environment.yml`

> [!IMPORTANT]
> This file is where you'll set up your infrastructure configuration such as:
>    - vCenter networks
>    - vCenter datastores
>    - vCenter templates locations
>    - Default virtual machine CPU and RAM settings
>    - LDAP server settings
>    - … _and more_

Simply put, fill it out with your infrastructure info, and you're almost done … _read below ..._

### Credentials
The accounts for the infrastructure components (`VMWare`, `Men & Mice`, `LDAP`) are being set through the following variables:

- `VC_USERNAME` (_vCenter_)
- `LDAP_USERNAME` (_LDAP_)
- `MM_USERNAME` (_Men & Mice_)

> [!TIP]
> It's highly recommended using a _service account_ rather than a regular user account when accessing these components.
> This is, an account that is **not** bound to a regular user.

<br />

#### Permissions
These accounts need to have a certain set of privileges in order to provision and terminate resources.

Here's a breakdown of the required permissions:

Component    | Privileges  |
:----------: | :---------: |
VMWare vSphere | Folder > Create folder<br />Folder > Delete Folder<br />Resource > Assign virtual machine to resource pool<br />Virtual machine > Configuration > CPU count<br />Virtual machine > Configuration > Memory<br />Virtual machine > Extend virtual disk<br />Virtual machine > Interaction > Device connection<br />Virtual machine > Interaction > Power Off<br />Virtual machine > Interaction > Power On<br />Virtual machine > Inventory > Create from existing<br />Virtual machine > Inventory > Remove<br />Virtual machine > Provisioning > Customize<br />Virtual machine > Provisioning > Deploy template<br />Virtual machine > Provisioning > Modify customization specifications<br />Virtual machine > Provisioning > Read customization specifications<br />Virtual machine > Configuration > Modify device settings<br />Virtual machine > Configuration > Settings<br />Network > Assign network |
Men & Mice  | DNS > Create record<br />DNS > Modify record<br />DNS > Remove record<br />IP address > Assign<br />IP address > Remove  |
LDAP   | Computer objects > Delete   |


#### Secrets
You must provide your infrastructure credentials to make the whole process work, and even though the `vars/environment.yml` file "declare" the variables that store these credentials - **THEY SHOULD NEVER BE PUT IN THERE IN PLAIN TEXT**

Instead, set the credentials variables as an _environment variables_ or via the _command line_.

The credentials variables are:
- `VC_PASSWORD`       (_vCenter_)
- `LDAP_PASSWORD`   (_LDAP_ or _Active Directory_)
- `MM_PASSWORD`       (_Men & Mice_)

> [!TIP]
> You can always use an external _Secrets Management_ tool such as Hashicorp Vault, CyberArk, Bitwarden, … to host these secrets, and then load them through automation tools like Jenkins or AWX.

<br />

## How to run this?
1. Get `Ansible` installed on the box meant to run the _playbooks_ ([How to install Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html))
   
2. On your command line:

* Install the Python modules dependencies:

```
# pip install -r requirements.txt
```

* To provision a new virtual machine:

```
# ansible-playbook provision_vm.yml
```

* To terminate an existing virtual machine:

```
# ansible-playbook terminate_vm.yml
```

3. In either case, in order to set your infrastructure credentials or override any of the settings in the `vars/environment.yml` file:
```
# ansible-plabook <provision_vm.yml|terminate_vm.yml> -e "VC_PASSWORD=<Your vCenter password> LDAP_PASSWORD=<Your LDAP password> MM_PASSWORD=<Your Men & Mice password> NET_DOMAIN=<Your Domain>"
```
