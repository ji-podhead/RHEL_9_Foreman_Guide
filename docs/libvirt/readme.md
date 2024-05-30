
 | [Knowledge Base](https://ji-podhead.github.io/RHEL_9_Foreman_Guide/knowledge%20base)| [Install](https://ji-podhead.github.io/RHEL_9_Foreman_Guide/installation%20(katello%2Cdiscovery%2Cdhcp%2Ctftp)) | [Discovery and Provisioning](https://ji-podhead.github.io/RHEL_9_Foreman_Guide/discovery%20and%20provisioning) | [libvirt](https://ji-podhead.github.io/RHEL_9_Foreman_Guide/libvirt) | [proxmox](https://ji-podhead.github.io/RHEL_9_Foreman_Guide/proxmox) | [diskless pxe-boot using zfs](https://ji-podhead.github.io/RHEL_9_Foreman_Guide/diskless_pxe_using_zfs) |


---

# libvirt
> - we gonna use root for KVM here,otherwise we need a kvm-user like this:
>  >```Bash
>  ># usermod -a -G libvirt _non_root_user_
>  >``` 
>  -  instead we will use this user: `root@kvm.mapping.com`, disable root-ssh login and login via local root password later on

## install

```Bash
$ su root
```

***create the folders needed for libvirt and the ssh keys***

```Bash
# mkdir /usr/share/foreman/.ssh
```
> -  ***the user needs to be foreman and it should be fully writable:***
>```Bash
># chmod 700 /usr/share/foreman/.ssh
># chown foreman:foreman /usr/share/foreman/.ssh
>```

-  (not sure if that was required)

> ```Bash
> # mkdir /usr/share/foreman/.cache
> # mkdir /usr/share/foreman/.cache/libvirt
> # mkdir /usr/share/foreman/.cache/libvirt/virsh
> # chown foreman:foreman /usr/share/foreman/.cache/libvirt/virsh
> # chmod 700 -R /usr/share/foreman/.cache 
> # chown foreman:foreman /usr/share/foreman/.cache
> ```

***install libvirt:***
```Bash
# dnf install qemu-kvm libvirt virt-install virt-viewer
```
```Bash
# for drv in qemu network nodedev nwfilter secret storage interface; do systemctl start virt${drv}d{,-ro,-admin}.socket; done
```


***validate:***
```Bash
# virt-host-validate
```
> -  *If all virt-host-validate checks return a PASS value, your system is prepared for creating VMs.*
>    - **see the red hat guide [Chapter 2. Enabling virtualization](https://access.redhat.com/documentation/de-de/red_hat_enterprise_linux/9/html/configuring_and_managing_virtualization/assembly_enabling-virtualization-in-rhel-9_configuring-and-managing-virtualization) for troubleshooting**

***enable and start libvirt:***
 ```Bash
# systemctl start libvirtd
```

 
- ***install virtmanager: *(optional)****
```Bash
# virt-manager
```

**check if the Virtual Bridge 0" interface  was created**

> ![virbr0](https://github.com/ji-podhead/RHEL_9_Foreman_Guide/blob/main/img/virbr0_kvm.png?raw=true)
```Bash
# ifconfig
```
>```
>enp2s0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
>        inet 192.168.2.100  netmask 255.255.255.0  broadcast 192.168.2.255
>...
>virbr0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
>        inet 192.168.122.1  netmask 255.255.255.0  broadcast 192.168.122.255
>...
>```
> - the ip of the Interface we are looking for: `192.168.122.1`
- [what is virbr0?](https://askubuntu.com/questions/246343/what-is-the-virbr0-interface-used-for)
> - *The virbr0, or "Virtual Bridge 0" interface is used for NAT (Network Address Translation). It is provided by the libvirt library, and virtual environments sometimes use it to connect to the outside network.*
> - whether you need to create a network bridge with virbr0 depends on your specific networking requirements and how you intend to manage network connections for your virtual machines (VMs).
> - In many setups, especially those involving libvirt and virtualization management tools like Foreman, a default bridge (virbr0) is often automatically created and managed by these systems. The virbr0 bridge is typically configured to allow VMs managed by libvirt to communicate with external networks, acting as a gateway for them.
> - However, if you have specific networking needs that require custom configurations beyond what virbr0 offers, such as bonding, VLAN tagging, or other advanced features, you might choose to manually create and configure a network bridge yourself.


---
## config


***add a host mapping***
> - edit the `/etc/hosts` file and add a mapping for our libvirt service
>   ```Bash
>	  ... 
>    192.168.2.100 cc.speedport.ip     # NIC`s main Ip used for this mapping - remember we had range of 100 
>    1192.168.122.1 kvm.mapping.com   # mapping for the virtual NIC we just created called vibr0
>   ```


***edit `/etc/ssh/sshd_config`:***
>```
>...
>Include /etc/ssh/sshd_config.d/*.conf
>PermitRootLogin yes
>```
> **the tricky part here is:**
> - we permit root login via ssh, but `we use the root user for KVM`
> - i think the reason why `PermitRootLogin yes` doesnt work is either the kvm-user, or the foreman user
> - both users dont have a pass, nor are there in the sudoers file
>    - but anway blocking root ssl login is best practise, but i still dont know the true cause
>    - so we just accept this for now and be happy that it works
- **dont forget to restart sshd!** 



***login to foreman:***
```Bash
# su foreman -s /bin/bash
```
***add ssh key:***
```Bash
bash-5.1$ ssh-keygen
```
***copy the key `(thats where we need root)`:***
```Bash
bash-5.1$ ssh-copy-id root@kvm.mapping.com
```

>```
>  ...
>  root@kvm.mapping.com's password:  <<------- ROOT
>  Number of key(s) added: 1
>  Now try logging into the machine, with:   "ssh 'root@kvm.mapping.com'"
>  and check to make sure that only the key(s) you wanted were added.
>```

***try the ssh connection:***
```Bash
 bash-5.1$ 'root@kvm.mapping.com'
```

***test the kvm-hypervisor connection:***
```Bash
bash-5.1$ virsh -c qemu+ssh://root@kvm.mapping.com/system
```
>```
> Willkommen bei virsh, dem interaktiven Virtualisierungsterminal.
> Tippen Sie:  'help' für eine Hilfe zu den Befehlen
>      'quit' zum Beenden
> virsh # 
>```

***exit the shell:***
```Bash
bash-5.1$ exit
```



***try to add the libvirt compute resource in foreman:***
> - open the dashboard, and try to add a computeresource like this:
> ![adding_computeresource](https://github.com/ji-podhead/RHEL_9_Foreman_Guide/blob/main/img/add_libvirt_computeresource.png?raw=true) 
> -	I had to restart my computer at before that because the libvirtd-admin.socket service stopped
>   > - you can check that by using systemctl:
>   >```Bash
>   >     # systemctl status libvirtd
>   >```
>   >```
>   >     ● libvirtd.service - libvirt legacy monolithic daemon
>   >     Loaded: loaded (/usr/lib/systemd/system/libvirtd.service; disabled; preset>
>   >     Active: active (running) since Mon 2024-05-27 16:21:53 CEST; 1s ago
>   >     TriggeredBy:
>   >       ● libvirtd-admin.socket
>   >       ● libvirtd-ro.socket
>   >	      ● libvirtd.socket
>   >```


---


## Creating and Configuring a Network Bridge on Linux Using nmcli ***(OPTIONAL)***


> The commands you executed are part of the process to create and configure a network bridge on a Linux system. This setup allows virtual machines (VMs) to communicate directly with the physical network, as if they were directly connected to the network via a physical network interface. Here's a comprehensive guide translated into English and formatted in Markdown:

***Step 1: Create a Network Bridge***
```Bash
bash sudo nmcli conn add type bridge con-name br0 ifname br0
```
> - This command creates a new network bridge named `br0`. A network bridge acts like a virtual switch, connecting multiple network interfaces, allowing traffic to be forwarded from one side of the network to another without passing through the physical device it arrived on.

***Step 2: Add a Physical Interface as a Slave to the Bridge***
```Bash
bash sudo nmcli conn add type ethernet slave-type bridge con-name bridge-br0 ifname enp2s0 master br0
```
> - With this command, the physical network interface `enp2s0` is added as a slave to the bridge `br0`. This connects the physical interface with the bridge, routing traffic passing through the bridge over the physical interface `enp2s0`.

***Step 3: Activate the Bridge***
```Bash
bash sudo nmcli conn up br0
```
> - This command activates the bridge `br0`, enabling it to function and forward traffic between connected interfaces.

***Step 4: Assign an IP Address to the Bridge (Optional)***
```Bash
bash sudo nmcli conn modify br0 ipv4.addresses "192.168.200.100/24" ipv4.method manual sudo nmcli conn up br0
```
> - These steps are optional and serve to assign a specific IP address to the bridge or virtual machine. In this example, the bridge `br0` is assigned the address `192.168.200.100` within the subnet `192.168.200.0/24`. After assigning the IP address, the bridge is reactivated to ensure the changes take effect.

> - This configuration enables virtual machines running over the bridge `br0` to communicate directly with the physical network, as if they were directly connected to the network via the physical network interface `enp2s0`. This setup is particularly useful for creating an isolated environment for virtual machines while still providing access to the physical network.



 | [Knowledge Base](https://ji-podhead.github.io/RHEL_9_Foreman_Guide/knowledge%20base)| [Install](https://ji-podhead.github.io/RHEL_9_Foreman_Guide/installation%20(katello%2Cdiscovery%2Cdhcp%2Ctftp)) | [Discovery and Provisioning](https://ji-podhead.github.io/RHEL_9_Foreman_Guide/discovery%20and%20provisioning) | [libvirt](https://ji-podhead.github.io/RHEL_9_Foreman_Guide/libvirt) | [proxmox](https://ji-podhead.github.io/RHEL_9_Foreman_Guide/proxmox) | [diskless pxe-boot using zfs](https://ji-podhead.github.io/RHEL_9_Foreman_Guide/diskless_pxe_using_zfs) |
