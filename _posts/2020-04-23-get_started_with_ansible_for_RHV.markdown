---
layout: single
title:  "Let’s get started with Ansible for RHV/oVirt!"
date:   2019-04-23 10:26:07 +0100
categories: ansible automation linux rhv ovirt virtualization
---
Have you ever think how to automate your everyday tasks on your RHV/oVirt infrastructure?

In this post we will guide you how to start your RHV/oVirt automation in just 15 min!

Let’s get started!

Red Hat Virtualization/oVirt includes support for the Ansible automation tool.
Ansible can be used to configure systems, deploy software, and perform rolling updates. Ansible provides modules that allow you to automate RHV post-installation tasks such as data center setup and configuration, managing users,  virtual machine operations and many more.

There are multiple reasons to choose Ansible if you need an automation tool for RHV/oVirt:
It’s easier to learn than existing SDKs or Rest APIs.
Even with limited programming knowledge, you will  find Ansible easier to use and get started than existing tools.

Lightweight and scalable.

Ansible is agentless and simply requires an SSH connection with the target systems. Ansible requires Python which is widely used in Linux distros.

Automatically installed with RHV/Ovirt 4.1 and later.
Ansible is automatically installed with Red Hat Virtualization Manager 4.1+ and oVirt Manager 4.1+, but it can also be installed on a separate machine using the RHV/oVirt repositories.

Ansible Roles are available to help configure and manage various parts of the RHV infrastructure.

Red Hat provides multiple roles in the ovirt-ansible-roles package that can be used to manage various parts of the RHV infrastructure, which makes using Ansible even easier.

## What you need to start

To start you RHV/oVirt infrastructure automation you need to make sure your Ansible host is equipped with a few things:

* Ansible Engine version >= 2.2
* python >= 2.7
* ovirt-engine-sdk-python >= 4.2.4

Most oVirt Ansible modules were introduced in Ansible Engine started from the version 2.2. Having that in mind it is critical to start with proper Ansible Engine version as most of  the modules we will ever use are shipped with Engine itself.
Python version is just a pure Ansible prerequisite. For most of usecases and modules python 2.7 is the minimum required to have on Ansible host.

The last but not least is `ovirt-engine-sdk-python` package which needs to be installed on Ansible host.
The software development kit is an interface for the Red Hat Virtualization/oVirt  REST API. As such, you must use the version of the software development kit that corresponds to the version of your Red Hat Virtualization/oVirt  environment. For example, if you are using Red Hat Virtualization 4.2, you must use the version of the software development kit designed for 4.2.
Having mention that, it is a good opportunity to stop for a while and explain how exactly Ansible works with platforms such as RHV/oVirt.

Lets take a look at the picture below:
As you can see there is a significant difference when it comes to playbook execution between “standard” OS based approach and platform such as Red Hat Virtualization / oVirt.

In short words our YAML code (playbook) is translated into a python code  and then python is translated into API calls to the platform, RHV/oVirt in this case.
Thanks to that the only prerequisites we need to take care of are placed on Ansible host. We do not need to change anything on RHV side to make it work!

How to install `ovirt-engine-sdk-python`

Keep in mind this needs to be done on Ansible host.

On Fedora >= 24 and CentOS 7

The SDK can be installed using the RPM packages
provided by the oVirt project.

To do so install the oVirt release package:

```bash
dnf install http://resources.ovirt.org/pub/yum-repo/ovirt-release42.rpm
```

Then install the SDK packages.

```bash
dnf install python-ovirt-engine-sdk4
```

For other operating systems (and also for Fedora and CentOS) you can
install the SDK using the `pip` command, which will download the source
from [PyPI][pip-url], build and install it.

On Red Hat Enterprise Linux 7

The SDK can be installed using the RPM packages provided by the RHV repos.
Enable the required channels:

```bash
subscription-manager repos --enable=rhel-7-server-rpms
subscription-manager repos --enable=rhel-7-server-rhv-4.2-rpms
```

Install the required packages:

```bash
yum install python-ovirt-engine-sdk4
```

Let’s Test It

Now when we have everything installed and properly set up there is a time to test and see if our Ansible host is able to communicate with RHV/oVirt environment and finally do some automation tasks.

Before we finally start our first Ansible playbook against RHV you need to know how to authenticate to your RHV/oVirt platform using Ansible.

To make it work we use `ovirt_auth` module.
This module authenticates to oVirt/RHV engine and creates SSO token, which should be later used in all other oVirt/RHV modules, so all modules don’t need to perform login and logout. This module returns an Ansible fact called `ovirt_auth`. Every module can use this fact as auth parameter, to perform authentication.

Knowing that now we can create two files containing necessary variables.

First one will be `rhv_vars.yml` which Contains variables needed to connect to the RHV Manager

```bash
cat rhvm_vars.yml
```

```yaml
---
rhvm_fqdn: rhvm.example.com
rhvm_user: admin@internal
rhvm_cafile: /etc/pki/ovirt-engine/ca.pem
```

Second one will be vault encrypted file with Red Hat Virtualization Manager password (rhvm_password).

```bash
cat password.yml
```

```yaml
---
rhvm_password: myrhvmpassword
```

As it is not a good idea to store passwords in plain text variable file that is why we will encrypt it using ansible provided vault mechanism.

```bash
$ ansible-vault encrypt password.yml
New Vault password:
Confirm New Vault password:
```

Now it is the time to put everything together and compose our final playbook.
This playbook will create VM from a template previously prepared and available on out test environment.

```bash
cat rhv_vm_create.yml
```

```yaml
---
- name: Create VM on RHV
  hosts: localhost
  connection: local
  gather_facts: false

  vars_files:
    - rhv_vars.yml
    - password.yml

  pre_tasks:
    - name: Login to oVirt
      ovirt_auth:
        hostname: "{{ rhvm_fqdn }}"
        username: "{{ rhvm_user }}"
        password: "{{ rhvm_password }}"
        ca_file: "{{ rhvm_cafile | default(omit) }}"
        insecure: "{{ rhvm_insecure | default(true) }}"
      tags:
        - always

  vars:
    datacenter: MyRHV
    cluster: MyRHVCluster
    template: rhel7-template
    vm_memory: 1GiB
    vm_name: test-vm01


  tasks:
    - name: Create and run VM from template
      ovirt_vms:
       auth: "{{ ovirt_auth }}"
        name: "{{ vm_name }}"
        template: "{{ template }}"
        cluster: "{{ cluster }}"
        memory: "{{vm_memory}}"
        high_availability: true
        state: present
        wait: yes

  post_tasks:
    - name: Logout from oVirt
      ovirt_auth:
        state: absent
        ovirt_auth: "{{ ovirt_auth }}"
      tags:
        - always
```

Now let’s take a closer look at some specific parameters we put in our playbook.

## Inventory and connection method

As we described previously how ansible works with RHV/oVirt we do not need to worry about inventory file here. This is because our playbook will be executed on Ansible host.

```bash
cat rhv_vm_create.yml
```

```yaml
---
- name: Create VM on RHV
  hosts: localhost
  connection: local
  gather_facts: false
```

## Variables

In our playbook you can find variables loaded from two files `rhv_vars.yml` and `passwod.yml` we prepared previously to fulfill `ovirt_auth` module needs.

The second set of variables were put in playbook itself to deliver all needed values for `ovirt_vm` module we use to manage VMs (create, delete, stop, start, etc.) on RHV/oVirt platform.

```bash
cat rhv_vm_create.yml
```

```yaml
---
- name: Create VM on RHV
  hosts: localhost
  connection: local
  gather_facts: false

  vars_files:
    - rhv_vars.yml
    - password.yml


...


   vars:
    datacenter: MyRHV
    cluster: MyRHVCluster
    template: rhel7-template
    vm_memory: 1GiB
    vm_name: test-vm01

...
```

## Pre and Post Tasks

As you can see also we have sections with pre and post tasks in our playbook. They are responsible for generating SSO token using `ovirt_auth` module (`pre_tasks` section) and at the end revoke token when all needed tasks were completed (`post_tasks` section).

Please pay attention we also introduced tags: always to make sure these tasks will always be run.

```bash
cat rhv_vm_create.yml
```

```yaml
---
- name: Create VM on RHV
  hosts: localhost
  connection: local
  gather_facts: false

...

  pre_tasks:
    - name: Login to oVirt
      ovirt_auth:
        hostname: "{{ rhvm_fqdn }}"
        username: "{{ rhvm_user }}"
        password: "{{ rhvm_password }}"
        ca_file: "{{ rhvm_cafile | default(omit) }}"
        insecure: "{{ rhvm_insecure | default(true) }}"
      tags:
        - always

...


  post_tasks:
    - name: Logout from oVirt
      ovirt_auth:
        state: absent
        ovirt_auth: "{{ ovirt_auth }}"
      tags:
        - always
```

Now let’s start our playbook:

```bash
ansible-playbook --ask-vault-pass rhv_vm_create.yml
```

## Useful links

[ovirt_vm][ovirt_vm_url] – Module to manage Virtual Machines in oVirt/RHV

[ovirt_auth][ovirt_auth_url] – Module to manage authentication to oVirt/RHV

[All RHV/oVirt Ansible modules][ovirt_modules_url]

[RHV Python SDK Guide][rhv_py_sdk_url]

[pip-url]: https://pypi.python.org/pypi
[ovirt_vm_url]: https://docs.ansible.com/ansible/latest/modules/ovirt_vm_module.html#ovirt-vm-module
[ovirt_auth_url]: https://docs.ansible.com/ansible/latest/modules/ovirt_auth_module.html
[ovirt_modules_url]: https://docs.ansible.com/ansible/latest/modules/list_of_cloud_modules.html#ovirt
[rhv_py_sdk_url]: https://access.redhat.com/documentation/en-us/red_hat_virtualization/4.2/html/python_sdk_guide/index
