# RHEL9 Foreman Guide
> In this Guide i will show you how to install Forman with puppet, katello and discovery plugin.
> You will also learn how to install and setup DHCP- and TFTP-Server.
> I will also show you how to setup Foreman and how to use the Foreman Boot Image via PXE.
> You will be ready to discover and provision your physical servers and workstations after following this Guide.

I decided to group the tutorials, rather than creating a huuuuge file.

## 1. [Knowledge Base]()
> - here we explain:
>	-  how tftp and dhcp works 
> 	-  how the pxe boot process works
>	-  how the foreman smartproxy works   

## 2. [Install Foreman (with katello, discovery-plugin, and local dhcp/tftp)]()
> - just the installation process

## 3. [Discovery and Provisioning]()
> - we discover our host using the Boot Image
> - we set up Hostgroups, subnets, etc
> - we finally provision our discovered host

## 4. [install and setup libvirt in Foreman]()
> - we install libvirt
> - we setup libvirt as compute resource
> - boot intoo container/vm

## 5. Proxmox
>  - we install proxmox via libvirt using foreman
>  - we setup proxmox as a compute resource
>  - we boot intoo container/vm
