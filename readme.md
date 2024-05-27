
# RHEL9 Foreman Guide

> - In this Guide i will show you how to install Forman with puppet, katello and discovery plugin.
> - You will also learn how to install and setup DHCP- and TFTP-Server.
> - I will also show you how to setup Foreman and how to use the Foreman Boot Image via PXE.
> - You will be ready to discover and provision your physical servers and workstations after following this Guide.


 
## 1. [Knowledge Base](https://ji-podhead.github.io/RHEL_9_Foreman_Guide/knowledge%20base)

> - here we explain:
>	-  how tftp and dhcp works 
> 	-  how the pxe boot process works
>	-  how the foreman smartproxy works   

## 2. [Install Foreman (with katello, discovery-plugin, and local dhcp/tftp)](https://ji-podhead.github.io/RHEL_9_Foreman_Guide/installation%20(katello%2Cdiscovery%2Cdhcp%2Ctftp))
> - just the installation process

## 3. [Discovery and Provisioning](https://ji-podhead.github.io/RHEL_9_Foreman_Guide/discovery%20and%20provisioning)
> - we discover our host using the Boot Image
> - we set up Hostgroups, subnets, etc
> - we finally provision our discovered host

## 4. [install and setup libvirt in Foreman](https://ji-podhead.github.io/RHEL_9_Foreman_Guide/libvirt)
> - we install libvirt
> - we setup libvirt as compute resource
> - boot intoo container/vm

## 5. Proxmox
>  - we install proxmox via libvirt using foreman
>  - we setup proxmox as a compute resource
>  - we boot intoo container/vm

# Roadmap
- libvirt
- proxmox
- lifecycle management with puppet and katello
- cicd with ansible, terraform and packer
- salt and k8s
---
***the original version of the guide can be found here:*** [original](https://ji-podhead.github.io/RHEL_9_Foreman_Guide/original)
> - I decided to group the tutorials, rather than creating a huuuuge file.

---
