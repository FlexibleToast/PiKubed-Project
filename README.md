# PiKubed

## What is this?

This is the working directory for all the ansible and scripts I use for my PiKubed project. PiKubed is my adventure into using Kubernetes on a cluster of four Raspberry Pi 4s. You can find more about the project at quick markdown site I'm using for notes [here](https://pikubed.com).

### Created So Far

All playbooks expect an inventory file that has a master and workers group. If you want a virtual IP for high availability access you can assign each host a `vip_priority` which will be used when the configure.yml playbook is run. For example my inventory file looks like:

```textfile
[pikubed_cluster:children]
master
workers

[master]
pikubed-m0  vip_priority=110

[workers]
pikubed-0  vip_priority=109
pikubed-1
pikubed-2
```
#### setup-hosts.yml

I'm using the CentOS 7 image available for the Raspberry Pi 4. With this image there is a default root user and ssh access. Therefore after flashing the SD cards with the image this playbook is run to setup all hosts to be prepared for the rest of the procedure. The following is performed:

- Update all packages

- If packages were updated the host is rebooted

- Expand the root filesystem to the entire SD card

- Change the root user password

- Add an ssh key for the root user if specified

- Create a user for Ansible

- Copy Ansible user's ssh key if specified

- Change the hostnames to match the inventory

#### configure.yml

This playbook handles the initial setup of the Raspberry Pis. The following is performed:

- System packages updated

- Creates `/etc/hosts` file that includes all cluster members on all hosts

- Moves `/tmp` to `tmpfs` to minimize SD card writes

- Installs log2ram to minimize SD card writes

  - You can use `log_folder_size` variable to adjust size in mb the size of the log folder in ram

- Installs k3s on master

- Adds workers to cluster using node-token

- Fetches .kube/config file and saves it as `k3s.yaml` on localhost

- Installs and configures a VIP for high availability access

#### upgrade.yml

Upgrades the master server, rejoins workers, restarts k3s service

#### reboot.yml

Reboots all nodes, one node at a time
