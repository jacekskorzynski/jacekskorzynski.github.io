---
layout: single
title:  "How to use Ansible Roles for oVirt/RHV"
date:   2019-07-20 18:26:07 +0100
categories: ansible automation linux rhv ovirt virtualization
---

Last time we spent some time to understand how Ansible works with RHV/oVirt infrastructure. We discussed all necessary packages and tools we need to start our first automation task. Then finally we were able to write and execute our first playbook against our demo RHV infrastructure.

That was the last time.

In this post we will concentrate on what has been already prepared. We will discover various sources where already prepared and ready to use roles wait for us.

So let’s get started.

## First of all let me explain to you what is an Ansible role

According to Ansible documentation: “Roles are ways of automatically loading certain vars_files, tasks, and handlers based on a known file structure. Grouping content by roles also allows easy sharing of roles with other users.”

Below you can find playbook structure example including roles structure:

```bash
site.yml
webservers.yml
fooservers.yml
roles/
   common/
     tasks/
     handlers/
     files/
     templates/
     vars/
     defaults/
     meta/
   webservers/
     tasks/
     defaults/
     meta/
```

Roles expect files to be in certain directory names. Roles must include at least one of these directories, however it is perfectly fine to exclude any which are not being used. When in use, each directory must contain a main.yml file, which contains the relevant content:

* **tasks** - contains the main list of tasks to be executed by the role.
* **handlers** - contains handlers, which may be used by this role or even anywhere outside this role.
* **defaults** - default variables for the role
* **vars** - other variables for the role
* **files** - contains files which can be deployed via this role.
* **templates** - contains templates which can be deployed via this role.
* **meta** - defines some meta data for this role. See below for more details.

Basically speaking you can treat the role as a playbook without your special settings like: users, passwords, inventory source etc.

You can also treat Ansible's role as a LEGO block.

One LEGO block = One Ansible role

Multiple Ansible roles put together in the right order can compose complex playbook which can touch different infrastructure platforms, operating systems, application installation tasks and so on.

You can also easily share your roles between different Ansible projects.

If you would like to dig a little bit deeper into this topic please read [Ansible Documentation][ansible-doc].

The second thing worth  mentioning is **Ansible Galaxy**.

[Ansible Galaxy][ansible-galaxy] is a free site for finding, downloading, rating, and reviewing all kinds of community developed Ansible roles and can be a great way to get a jumpstart on your automation projects.
Ansible comes with included tools to speed up your work with Galaxy.
One of them is an `ansible-galaxy` client. The Galaxy client allows you to download roles from Ansible Galaxy, and also provides an excellent default framework for creating your own roles.

As you can see on the picture below you can find there pretty  rich repository of oVirt/RHV related roles.

## Let’s have a look how to start with RHV/oVirt Ansible roles

oVirt maintains multiple Ansible roles that can be deployed to easily configure and manage various parts of the oVirt infrastructure.

Below you can find github link:

[oVirt Ansible Roles][ovirt-roles]

![oVirt Ansible Galaxy Roles](/assets/post002/galaxy01.png)

Currently there are available following Ansible roles:

* **oVirt.cluster-upgrade** - easily upgrade your oVirt clusters, host by host.
* **oVirt.disaster-recovery** - plan, failover and failback oVirt in Disaster Recovery scenarios.
* **oVirt.engine-setup** - setup your oVirt Engine via Ansible.
* **oVirt.infra** - setup a complete oVirt setup (data centers, clusters, hosts, networks...) via this role.
* **oVirt.image-template** - easily create VM templates (via Glance or QCOW2 download)
* **oVirt.manageiq** - install and configure a ManageIQ (or CloudForms) VM appliance on your oVirt!
* **oVirt.repositories** - set up the required oVirt repositories on your hosts.
* **oVirt.vm-infra** - configure a complete VM setup (create and configure VMs and their properties)
* **oVirt.v2v-conversion-host** - define a host as a target for VMware to oVirt migration.
* **oVirt.hosted_engine_setup** - setup your oVirt Hosted-Engine via Ansible.
* **oVirt.shutdown_env** - shutdown the whole environment in a clean and ordered way.

## Installing oVirt/RHV Ansible Roles

You can install Ansible roles for Red Hat Virtualization from the Red Hat Virtualization Manager repository or  from the oVirt Manager repository if you use the oVirt platform.

To achieve that use the following command to install all roles:

```bash
yum install ovirt-ansible-roles
```

Run following command to install specific role:

```bash
yum install ovirt-ansible-infra
```

We are going to install the Ansible roles on the Manager machine.

By default the roles are installed to `/usr/share/ansible/roles`.

The structure of the `ovirt-ansible-roles` package is as follows:

`/usr/share/ansible/roles` - stores the roles.

`/usr/share/doc/ovirt-ansible-roles/` - stores the examples, a basic overview, and the licence.

`/usr/share/doc/ansible/roles/role_name` - stores the documentation specific to the role.

The other way to grab RHV/oVirt Ansible Role is a `ansible-galaxy` client.

As we mentioned previously the `ansible-galaxy` command comes bundled with Ansible, and you can use it to install roles from Galaxy or directly from a git based SCM. You can also use it to create a new role, remove roles, or perform tasks on the Galaxy website.

The command line tool by default communicates with the Galaxy website API using the server address [https://galaxy.ansible.com]. Since the Galaxy project is an open source project, you may be running your own internal Galaxy server and wish to override the default server address. You can do this using the `–server` option or by setting the Galaxy server value in your `ansible.cfg` file.

## Installing Roles from Galaxy

Use the `ansible-galaxy` command to download roles from the Galaxy website.

```bash
ansible-galaxy install username.role_name
```

For example:

```bash
ansible-galaxy install ovirt.hosted_engine_setup
```

By default Ansible downloads roles to the first writable directory in the default list of paths

`~/.ansible/roles:/usr/share/ansible/roles:/etc/ansible/roles.`

This will install roles in the home directory of the user running `ansible-galaxy`.

You can override this by setting:
the environment variable ANSIBLE_ROLES_PATH in your session,
defining `roles_path` in an `ansible.cfg` file,  
using the `--roles-path` option.

## Running oVirt Ansible Roles

The following procedure guides you through creating and running a playbook that uses Ansible roles to configure Red Hat Virtualization. This example uses Ansible to connect to the Manager on the local machine and create a new data center.

**Prerequisites**
Ensure the roles_path option in `/etc/ansible/ansible.cfg` points to the location of your Ansible roles (`/usr/share/ansible/roles`).

Ensure that you have the `oVirt Python SDK` installed on the machine running the playbook.

### Configuring RHV / oVirt using Ansible Roles

1. Create a file in your working directory to store the Red Hat Virtualization Manager user password:

    ```bash
    cat passwords.yml
    ```

    ```yaml
    ---
    engine_password: redhatpassword
    ```

2. Encrypt the user password. You will be asked for a Vault password.

    ```bash
    ansible-vault encrypt passwords.yml
    New Vault password:
    Confirm New Vault password:
    ```

3. Create a file that stores the Manager details such as the URL, certificate location, and user.

    ```bash
    cat engine_vars.yml
    ```

    ```yaml
    ---
    engine_url: https://rhvm.example.local/ovirt-engine/api
    engine_user: admin@internal
    ```

4. Create your playbook. To simplify this you can copy and modify an example in `/usr/share/doc/ovirt-ansible-roles/` examples.

    ```bash
    cat rhv_infra.yml
    ```

    ```yaml
    ---
    - name: RHV infrastructure
    hosts: localhost
    connection: local
    gather_facts: false

    vars_files:
        # Contains variables to connect to the Manager
        - engine_vars.yml
        # Contains encrypted engine_password variable using ansible-vault
        - passwords.yml

    pre_tasks:
        - name: Login to RHV
        ovirt_auth:
            url: "{{ engine_url }}"
            username: "{{ engine_user }}"
            password: "{{ engine_password }}"
            ca_file: "{{ engine_cafile | default(omit) }}"
            insecure: "{{ engine_insecure | default(true) }}"
        tags:
            - always

    vars:
        data_center_name: newdatacenter
        data_center_description: newdatacenter
        data_center_local: false
        compatibility_version: 4.2

    roles:
        - ovirt-datacenters

    post_tasks:
        - name: Logout from RHV
        ovirt_auth:
            state: absent
            ovirt_auth: "{{ ovirt_auth }}"
        tags:
            - always
    ```

5. Run the playbook

    ```bash
    ansible-playbook --ask-vault-pass rhv_infra.yml
    ```

You have successfully used the ovirt-datacenters Ansible role to create a data center named newdatacenter.

[ansible-doc]: https://docs.ansible.com/
[ansible-galaxy]: https://galaxy.ansible.com/
[ovirt-roles]: https://github.com/oVirt/ovirt-ansible
