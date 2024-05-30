
**| [Knowledge Base](https://ji-podhead.github.io/RHEL_9_Foreman_Guide/knowledge%20base)|[Install](https://ji-podhead.github.io/RHEL_9_Foreman_Guide/installation%20(katello%2Cdiscovery%2Cdhcp%2Ctftp)) | [Discovery and Provisioning](https://ji-podhead.github.io/RHEL_9_Foreman_Guide/discovery%20and%20provisioning) | [libvirt](https://ji-podhead.github.io/RHEL_9_Foreman_Guide/libvirt) | [proxmox](https://ji-podhead.github.io/RHEL_9_Foreman_Guide/proxmox) |** 

---
***UNDER CONSTRUCTION!***
## proxmox ZFS tank
***add the disk’s wee need for the tank to our wm***
![add_disk](https://github.com/ji-podhead/RHEL_9_Foreman_Guide/blob/main/img/zfs1_kvm_add_disk.png?raw=true)
***create zfs called tank***
![create_tank](https://github.com/ji-podhead/RHEL_9_Foreman_Guide/blob/main/img/zfs2_creating_zfs.png?raw=true)
***create datasets for our zfs tank in proxmox shell:***
```Bash
# zfs create tank/backups
# zfs create tank/isos
# zfs create tank/diskstorage
```
***check it out:***
```Bash
# zfs list
# zpool list
```
***
***create the zfs storage directories***
![create_storage](https://github.com/ji-podhead/RHEL_9_Foreman_Guide/blob/main/img/zfs3_create_storage.png?raw=true)***upload a iso (optional)***
![upload_iso](https://github.com/ji-podhead/RHEL_9_Foreman_Guide/blob/main/img/zfs4_upload_iso.png?raw=true)***move the wm storage to zfs (optional):***
![move_storage](https://github.com/ji-podhead/RHEL_9_Foreman_Guide/blob/main/img/zfs5_move_wm_storage.png?raw=true)***create a backup for our wm using our zfs_back storage directory(optional)***
![backup](https://github.com/ji-podhead/RHEL_9_Foreman_Guide/blob/main/img/zfs6_wm_backup.png?raw=true)
## nfs
***in proxmox shell:***
```
# apt install nfs-common
# apt install nfs-kernel-server
# mkdir -p /mnt/shared_folder_on_nfs
# chmod -R 777 /tank/diskstorage
# chown -R nobody:nogroup /tank/diskstorage
```
***create zfs shared folder:***
```Bash
# zfs create tank/nfs_shared_folder
# zfs set sharenfs=on tank/nfs_shared_folder
```
***edit the exports file:***
```Bash
# nano /etc/exports
```
>```
>...
># /srv/nfs4/homes  gss/krb5i(rw,sync,no_subtree_check)
>#
>/proxmox.local:/tank/nfs_shared_folder *(rw,sync,no_subtree_check)
>```
***edit the fstab:***
```Bash
# nano /etc/fstab
```
>```
>...
>proc /proc proc defaults 0 0
>
>proxmox.local:/tank/diskstorage /mnt/shared_folder_on_nfs nfs auto 0 0
>```
***update Grub:***
```Bash
#sudo apt-get install --reinstall dracut
#dracut -f
```
***edit the wm-config:***
- this needs to be done in the machine that runs libvirt not inside proxmox
```Bash
# virsh edit <your_proxmox_wm>
```

> - add `<shareable/>` to the disk we added to create the zfs tank
> ```
>      <disk type='block' device='disk'>
>      <driver name='qemu' type='raw' cache='none' io='native' discard='unmap'/>
>      <source dev='/dev/sdc'/>
>      <target dev='sdc' bus='sata'/>
>      <shareable/>
>      <address type='drive' controller='0' bus='0' target='0' unit='2'/>
>      </disk>
>```


***we can mount the zfs tank thats is shared via nfs like this:***
```
# mount -t nfs 192.168.122.166:/<mountpoint> /mnt/shared_folder_on_nfs
```
---
**| [Knowledge Base](https://ji-podhead.github.io/RHEL_9_Foreman_Guide/knowledge%20base)|[Install](https://ji-podhead.github.io/RHEL_9_Foreman_Guide/installation%20(katello%2Cdiscovery%2Cdhcp%2Ctftp)) | [Discovery and Provisioning](https://ji-podhead.github.io/RHEL_9_Foreman_Guide/discovery%20and%20provisioning) | [libvirt](https://ji-podhead.github.io/RHEL_9_Foreman_Guide/libvirt) | [proxmox](https://ji-podhead.github.io/RHEL_9_Foreman_Guide/proxmox) |** 





