
# RHEL9 Foreman Guide

<div align="center">
      <img src="https://github.com/ji-podhead/ji-podhead/blob/main/logo.jpg?raw=true" align="right" width="150" />
</div>

> - In this Guide i will show you how to install Forman with puppet, katello and discovery plugin.
> - You will also learn how to install and setup DHCP- and TFTP-Server.
> - I will also show you how to setup Foreman and how to use the Foreman Boot Image via PXE.
> - You will be ready to discover and provision your physical servers and workstations after following this Guide.


 
## 1. [Knowledge Base](https://ji-podhead.github.io/RHEL_9_Foreman_Guide/knowledge%20base)

> - here we explain:
>	  -  how tftp and dhcp works 
>   -  how the pxe boot process works
>	  -  how the foreman smartproxy works
>   -  Lifecycle Management
>      - what is it?
>      - puppet & katello roles

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

## 5. [Proxmox](https://ji-podhead.github.io/RHEL_9_Foreman_Guide/proxmox)
>  - we install proxmox inside a vm using kvm&libvirt 
>  - we setup proxmox as a compute resource
## 6. [Install Foreman with external DHCP & DNS](https://ji-podhead.github.io/RHEL_9_Foreman_Guide/nestedVM_with_external_DHCP%26DNS)
>  - we install foreman inside a nested VM
>  - we set up our DHCP & DNS for Dynamic Updates using RNDC 
>  - we configure our DHCP to share its leases using omapi(HMAC-MD5) key and NFS
>  - we configure Foreman to manage our external DNS by importing the RNDC key
>  - we configure Foreman to manage our external DHCP by using remote-isc-key flag and our omapi key   
## 7. [diskless pxe-boot using zfs](https://ji-podhead.github.io/RHEL_9_Foreman_Guide/diskless_pxe_using_zfs) *(under construction)*
> - we create a zfs tank inside proxmox
> - we create a wm inside proxmox and move the storage to our zfs tank *(optional)*
> - we create a automatic backup-plan for the wm *(optional)*
> - we create a pxe template inside foreman to pxe-boot diskless using the zfs tank storage 

---

# Roadmap
- ~~libvirt~~ ✓
- ~~proxmox~~ ✓
- ~~diskless boot using zfs (incl. repo storage) and custom pxe/grub preset~~ ✓
- lifecycle management with puppet and katello
- cicd with ansible, terraform and packer
- salt, k8s and kubevirt
---
***the original version of the guide can be found here:*** [original](https://ji-podhead.github.io/RHEL_9_Foreman_Guide/original)
> - I decided to group the tutorials, rather than creating a huuuuge file.

---
