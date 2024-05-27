**| [Knowledge Base](https://ji-podhead.github.io/RHEL_9_Foreman_Guide/knowledge%20base)|[Install](https://ji-podhead.github.io/RHEL_9_Foreman_Guide/installation%20(katello%2Cdiscovery%2Cdhcp%2Ctftp)) | [Discovery and Provisioning](https://ji-podhead.github.io/RHEL_9_Foreman_Guide/discovery%20and%20provisioning) | [libvirt](https://ji-podhead.github.io/RHEL_9_Foreman_Guide/libvirt) | [proxmox](https://ji-podhead.github.io/RHEL_9_Foreman_Guide/proxmox) |** 

---

# libvirt
## install

```Bash
$su root
```
```Bash
# dnf install qemu-kvm libvirt virt-install virt-viewer
```
```Bash
for drv in qemu network nodedev nwfilter secret storage interface; do systemctl start virt${drv}d{,-ro,-admin}.socket; done
```


***validate:***
```Bash
# virt-host-validate
```
> -  *If all virt-host-validate checks return a PASS value, your system is prepared for creating VMs.*
>    - **see the red hat guide [Chapter 2. Enabling virtualization](https://access.redhat.com/documentation/de-de/red_hat_enterprise_linux/9/html/configuring_and_managing_virtualization/assembly_enabling-virtualization-in-rhel-9_configuring-and-managing-virtualization)** for troubleshooting**

**setupt the virtual NIC**
- check if it was installed before:
```Bash
# ifconfig
```
>```
>enp2s0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
>        inet 192.168.2.100  netmask 255.255.255.0  broadcast 192.168.2.255
>...
>```

```Bash
# sudo nmcli conn add type bridge con-name br0 ifname br0
# sudo nmcli conn add type ethernet slave-type bridge con-name bridge-br0 ifname enp2s0 master br0
# sudo nmcli conn up br0
```

>```
>Verbindung »br0« (02c66e55-068d-4aca-a03b-1559b32917cd) erfolgreich hinzugefügt.
>Verbindung »bridge-br0« (0cdcc86e-76dd-48e4-9c94-ebb2b68d7f55) erfolgreich hinzugefügt.
>Verbindung wurde erfolgreich aktiviert (master waiting for slaves) (Aktiver D-Bus-Pfad: /org/freedesktop/NetworkManager/ActiveConnection/27)
>```
- ***we assign an ip:***
```Bash
# sudo nmcli conn modify br0 ipv4.addresses "192.168.200.100/24" ipv4.method manual
# sudo nmcli conn up br0
```
>```
>Verbindung wurde erfolgreich aktiviert (master waiting for slaves) (Aktiver D-Bus-Pfad: /org/freedesktop/NetworkManager/ActiveConnection/28)
>```

**add a host mapping**
> - edit the `/etc/hosts` file and add a mapping for our libvirt service
>```Bash
>... 
>192.168.2.100 cc.speedport.ip     # NIC`s main Ip used for this mapping - remember we had range of 100 
>192.168.200.100 kvm.mapping.com   # mapping for the virtual NIC we just created called br0
>```





---
**| [Knowledge Base](https://ji-podhead.github.io/RHEL_9_Foreman_Guide/knowledge%20base)|[Install](https://ji-podhead.github.io/RHEL_9_Foreman_Guide/installation%20(katello%2Cdiscovery%2Cdhcp%2Ctftp)) | [Discovery and Provisioning](https://ji-podhead.github.io/RHEL_9_Foreman_Guide/discovery%20and%20provisioning) | [libvirt](https://ji-podhead.github.io/RHEL_9_Foreman_Guide/libvirt) | [proxmox](https://ji-podhead.github.io/RHEL_9_Foreman_Guide/proxmox) |** 
