



# Discovery and Provisioning
 
## Discovery
***Change PXE global temlate***
> set it to discovery
![settings](https://github.com/ji-podhead/RHEL_9_Foreman_Guide/blob/main/img/settings.png?raw=true)

***Add a subnet***
> - i use the maximum range here, but you dont need to if you have multiple hostgroups
>     - keep in  mind that this is limited by your dhcp-settings 
![subnet](https://github.com/ji-podhead/RHEL_9_Foreman_Guide/blob/main/img/subnet.png?raw=true)


***Build PXE default template***
> click on *A)*
![pxe defauflt](https://github.com/ji-podhead/RHEL_9_Foreman_Guide/blob/main/img/provisioning_templates.png?raw=true) 
> - also make sure to add the proxy to your subnet:
> ![subnet_proxy](https://github.com/ji-podhead/RHEL_9_Foreman_Guide/blob/main/img/subnet_proxy.png?raw=true)
>
***Boot a machine via PXE***
> i was to lazy to setup kvm and libvirt so i used my server here
> if everything works, you dont need to do anything: just let it run
![pxe_boot](https://github.com/ji-podhead/RHEL_9_Foreman_Guide/blob/main/img/first%20discovery.jpg?raw=true)

- if your subnet and settings are correct it should boot:
> ![pxe_boot](https://github.com/ji-podhead/RHEL_9_Foreman_Guide/blob/main/img/discovery_boot_process.jpg?raw=true)
> - the Boot Image will configure NIC manually:
	>  	- waiting time is quite long, but thise way boot wont fail  
> ![auto_discovery](https://github.com/ji-podhead/RHEL_9_Foreman_Guide/blob/main/img/discovery_auto_setup.jpg?raw=true) 
> 
> - the host gets discovered:
> ![discovery_finish](https://github.com/ji-podhead/RHEL_9_Foreman_Guide/blob/main/img/discovery_finish.jpg?raw=true)
> 
> - our discovered sost is ready to get provisioned:
> ![discovered_host](https://github.com/ji-podhead/RHEL_9_Foreman_Guide/blob/main/img/discovered_host.jpg?raw=true)
 ## Provisioning
> - we need a os:
> ![os](https://github.com/ji-podhead/RHEL_9_Foreman_Guide/blob/main/img/os_media.jpg?raw=true)
> > - it also needs a partition table:
> ![os_partition_table](https://github.com/ji-podhead/RHEL_9_Foreman_Guide/blob/main/img/os_partition.jpg?raw=true)
 - clone the Provisioning template:
> > click on B)
> > ![pxe defauflt](https://github.com/ji-podhead/RHEL_9_Foreman_Guide/blob/main/img/provisioning_templates.png?raw=true)
> > - we need to add a os to it:
> ![os_pxe_template](https://github.com/ji-podhead/RHEL_9_Foreman_Guide/blob/main/img/default_pxe_clone.jpg?raw=true)

 - add a Hostgroup
> > select our subnet:
> >![hostgroup](https://github.com/ji-podhead/RHEL_9_Foreman_Guide/blob/main/img/hostgroup.jpg?raw=true)
> - for simplicity i just choose my default os as the hostgroup’s os:
> ![hostgroup_os](https://github.com/ji-podhead/RHEL_9_Foreman_Guide/blob/main/img/hostgroup_os.jpg?raw=true)
 - now we can create a host from the machine that was discovered before
 > *you need to click on provision in the Discovered Host’s section*
 > - we select our hostgroup *main* which we created before:
> > ![create_host](https://github.com/ji-podhead/RHEL_9_Foreman_Guide/blob/main/img/provision_host.jpg?raw=true)
> - in the Host section we need to select our proxy and hostgroup:
> > ![host_section](https://github.com/ji-podhead/RHEL_9_Foreman_Guide/blob/main/img/provision_host_main.jpg?raw=true)
> - add a password to the host and finish the provision process:
> >![host_password](https://github.com/ji-podhead/RHEL_9_Foreman_Guide/blob/main/img/provision_host_os.png?raw=true)

