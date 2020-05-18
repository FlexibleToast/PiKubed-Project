# PiKubed

## What is this?

This is the working directory for all the ansible and scripts I use for my PiKubed project. PiKubed is my adventure into using Kubernetes on a cluster of four Raspberry Pi 4s. You can find more about the project at quick markdown site I'm using for notes [here](https://pikubed.com).

### Created So Far

All playbooks expect an inventory file that has a master and workers group. If you want a virtual IP for high availability access you can assign each host a `vip_priority` which will be used when the configure.yml playbook is run. For example my inventory file looks like:

```textfile
[pikubed_cluster:children]
masters
workers

[pikubed_cluster:vars]
ansible_python_interpreter=/usr/bin/python3 # This is used because Debian defualts to python2 still

[masters]
pikubed-m0  vip_priority=110
pikubed-m1  vip_priority=109
pikubed-m1  vip_priority=108

[workers]
pikubed-w0
```
#### vars.yml.template
Make a copy of this file called vars.yml and populate with the values for your environment. Note that the way the main playbook works it uses the root user, sets root's password, and can reboot the nodes at some later time. When the node reboots the password Ansible is using will no longer be valid and will fail to connect. This is okay as the play should be idempotent, just re-run main.yml and specify the new password. Another way around this is to assign the root user the same ssh key as Ansible uses.

#### main.yml
This is the playbook used to install your Kubernetes cluster. Right now there are two usable options, high availability and non-high availability. By default it is configured for high availability and tested on Raspbian. A lot of work was done using the CentOS 7 image so minimal changes should be required to get it to work with CentOS 7. It will do the following:

  - Update all hosts
  - Install required packages
  - Reboot nodes to ensure using latest updates
  - Expand the root filesystem to entire SD card
  - Update root user password and add ssh key
  - Create ansible user and ssh key if user is defined
  - Change hostname of machines to match inventory names
  - Create /etc/hosts including all node members
  - Move the /tmp directory to tmpfs
  - Install log2ram
  - Distribute ssh keys (Only needed if using k3sup method that is currently not working since I've coded it to use the embedded HA that does not work in k3s at this time)
  - Install keepalived for virtual IP
  - Install Galera (highly available MariaDB used for install-k3s-ha role creating an "external datastore")
  - Install k3s on all nodes

#### upgrade.yml NOT UPDATED YET HA

This was created when I was still using the non high availability default installation of k3s. The playbook needs to be updated.

#### reboot.yml

Reboots all nodes, one node at a time
