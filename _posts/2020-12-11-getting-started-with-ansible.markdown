---
layout: single
title:  "Getting Started with Ansible"
date:   2020-12-11 00:26:07 +0100
categories: ansible automation linux
---
Ansible is a great technology to automate your IT infrastructure, helps with application development process and do many more.

You are probably thinking how to start your automation journey with Ansible.

So lets get started!

## Installation

First of all lets install ansible.
You can do it on various different operating systems.
Here I would like to show you two different ways:

* Installation from repository on RHEL / CentOS / Fedora
* Python `pip` method

### RHEL / CentOS / Fedora

Before you start there is a need to setup proper repositories.
Ansible package is part of separate repository so you are not going to find it in default RHEL repos.
That is why we need to attach is before installation will start.

```bash
subscription-manager repos --enable ansible-2.9-for-rhel-8-x86_64-rpms
```

And afret that just simple (RHEL 8 / CentOS 8 / Fedora):

```bash
dnf install ansible
```

or on RHEL 7 / CentOS 7

```bash
yum install ansible
```

Let's make sure everything works:

```bash
ansible --version
```

```bash
ansible 2.9.16
  config file = /etc/ansible/ansible.cfg
  configured module search path = ['/home/jskorzyn/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3.9/site-packages/ansible
  executable location = /usr/bin/ansible
  python version = 3.9.1 (default, Dec  8 2020, 00:00:00) [GCC 10.2.1 20201125 (Red Hat 10.2.1-9)]
```

That is basically it. You have your ansible and you are ready to go.

### Python `pip` - Ansible 2.10

Ansible can be installed with `pip` Python package manager.

To make it work you just need pip to be present on your system.
If it is not there yet just follow procedure below:

```bash
curl <https://bootstrap.pypa.io/get-pip.py> -o get-pip.py
python get-pip.py --user
```

Then run start command below:

```bash
python -m pip install --user ansible
```

**Note:** If you have Ansible 2.9 or older installed, you need to use `pip uninstall ansible` first to remove older versions of Ansible before re-installing it.
{: .note}

You can find more examples in [Ansible Docs][ansible-docs]

### First Playbook

Now when Ansible is ready we can start with simple configuration and finally run our first playbook.

#### Inventory

Inventory is basically a list of endpoints you want to automate.
It can be either a manual inventory (like static file) or dynamic one.
Here I would like to concentrate on static one.

By default Ansible uses inventory comming form `/etc/ansible/hosts`

First of all lets create a folder where we are going to place all neede files.

```bash
mkdir ./first-playbook
cd ./first-playbook
```

[ansible-docs]: https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html
