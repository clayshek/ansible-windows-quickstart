# Ansible Windows Demo Vagrant Quickstart

Vagrant provisioning of an Ansible playground on top of Windows (Hyper-V). Built for Ansible demo, not for production purposes.

Vagrantfile will provision 3 machines by default:
- ansiblectl (Ubuntu 20.04) - Base Ubuntu image with Ansible installed
- wintest (Windows Server 2019)
- awx (Ubuntu 20.04) - Ubuntu VM with [AWX](ansible.com/products/awx-project) & all prerequisites (Docker, etc)

## Requirements:

- [Hyper-V: Installed & functional](https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/quick-start/enable-hyper-v), with a virtual switch that allows outbound Internet connectivity (for package downloads)
- [Vagrant installed](https://www.vagrantup.com/downloads)

> **Warning**: Default VM setup as follows. Ensure that you have adequate system resources, or only deploy a subset.
>
> ansiblectl & wintest: 1 GB RAM /ea
>
> awx: 4 GB RAM

## Usage:

Clone this repo. From within the repo directory, run: `vagrant up`. The only prompt should be asking which Hyper-V virtual switch to use. The Vagrant Hyper-V provider [can't figure this out yet](https://www.vagrantup.com/docs/providers/hyperv/limitations).

Minimum deployment is ansiblectl & wintest. These require less resources than AWX. To deploy just minimum:

`vagrant up ansiblectl wintest`

To deploy AWX: `vagrant up awx`

First run will be slower while box images are downloaded. On completion the virtual machines referenced above should be running in Hyper-V, with Ansible installed on 'ansiblectl' and basic starter files in /home/vagrant/ansible-wintest. Linux VM has SSH connectivity, Windows has RDP. IP addresses will be DHCP assigned. Watch "vagrant up" output for IPs, or use commands below to let Vagrant figure it out for you.

SSH to Linux VMs: `vagrant ssh ansiblectl` or `vagrant ssh awx`

RDP to Windows VMs: `vagrant rdp wintest`

Credentials for each VM will be `vagrant/vagrant`

The VMs also each have mDNS software (Avahi on Linux, Bonjour on Windows) installed to [help with name resolution](http://www.multicastdns.org/).

## Basic Ansible Commands:
- Run from Ansiblectl, inside the `ansible-wintest` directory.

Connectivity check: `ansible wintest -i inventory.yml -m win_ping`

Gather facts: `ansible wintest -i inventory.yml -m setup`

Run example included playbook to install Telegraf: `ansible-playbook -i inventory.yml plays/install_telegraf.yml`

## AWX:

Once the "AWX" VM provisioning is complete, the AWX console should be accessible at http://awx.local. On first access, the AWX database setup will take a few minutes, but once done, login credentials are `admin` and `password`.

As AWS is in a Docker container, and mDNS doesn't work well, inventory for wintest should be setup as "wintest", without the .local FQDN.

## Cleanup:

Destroy all VMs:  `vagrant destroy --force`  

Destroy specific VMs: `vagrant destroy vmname`

## Reference Docs:

- [Vagrant Docs](https://www.vagrantup.com/docs/index)
- [Ansible Windows Docs](https://docs.ansible.com/ansible/latest/user_guide/windows.html)
- [Available Windows Modules](https://docs.ansible.com/ansible/latest/collections/community/windows/index.html#plugins-in-community-windows)
- [Available Windows Community Modules](https://docs.ansible.com/ansible/latest/collections/community/windows/index.html#plugins-in-community-windows)
- [Install AWX on Ubuntu](https://computingforgeeks.com/how-to-install-ansible-awx-on-ubuntu-linux/)
- [AWX on GitHub](https://github.com/ansible/awx)
- [AWX Quickstart](https://docs.ansible.com/ansible-tower/latest/html/quickstart/index.html)
