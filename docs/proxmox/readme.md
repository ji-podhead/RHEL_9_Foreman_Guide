
| [Knowledge Base](https://ji-podhead.github.io/RHEL_9_Foreman_Guide/knowledge%20base)| [Install](https://ji-podhead.github.io/RHEL_9_Foreman_Guide/installation%20(katello%2Cdiscovery%2Cdhcp%2Ctftp)) | [Discovery and Provisioning](https://ji-podhead.github.io/RHEL_9_Foreman_Guide/discovery%20and%20provisioning) | [libvirt](https://ji-podhead.github.io/RHEL_9_Foreman_Guide/libvirt) | [proxmox](https://ji-podhead.github.io/RHEL_9_Foreman_Guide/proxmox) | [diskless pxe-boot using zfs](https://ji-podhead.github.io/RHEL_9_Foreman_Guide/diskless_pxe_using_zfs) |
---

# proxmox 
- in this section we will install proxmox in a wm
- we will create a self-signed ssl-cert and create the proxmox-computeresource inside foreman

## install
> - download the iso
> - create a new vm
> - install proxmox inside the vm
> 
> ![create_proxmox_vm](https://github.com/ji-podhead/RHEL_9_Foreman_Guide/blob/main/img/libvirt_create_proxmox_vm.png?raw=true)
> ![promox_install](https://github.com/ji-podhead/RHEL_9_Foreman_Guide/blob/main/img/libvirt_initial_proxmox_boot.png?raw=true)
> ![proxmox_finish](https://github.com/ji-podhead/RHEL_9_Foreman_Guide/blob/main/img/libvirt_proxmox_complete.png?raw=true)

> login via your local browser using "root" along with the password you set in installation-process

---

***add a host-mapping:***
> - edit /etc/hosts and add a mapping for the proxmox ip, so we can create a self-signed sll cert
> ```
> ...
> 192.168.122.1 kvm.mapping.com
> 192.168.122.166 my.proxmox-server.de
>```

## create a self-signed ssl-cert
 - we need this to configure proxmox-computeresource in foreman
> - otherwise foreman will give this **error:** ` 
>```
> ERF42-5577 [Foreman::Exception]: Failed to create Proxmox compute resource: 
> SSL_read:  unexpected eof while reading (OpenSSL::SSL::SSLError). 
> Either provided credentials or FQDN is wrong or your server cannot connect to Proxmox due to network issues.
>```
  - of course you can use letsencrypt with certmanager/trafik or buy a cert
 -  but thats to much for this tutorial,
 -  so we will just use **openssl** to create the cert with a few lines of code
 
***create a private key:***
```Bash
# openssl genpkey -algorithm RSA -out private_key.pem
```
***encrypt your private key:***
```Bash
# openssl rsa -in private_key.pem -out encrypted_private_key.pem
```
>```
> writing RSA key
>```

***create a csr:***
```Bash
# openssl req -new -key private_key.pem -out csr.pe
```

>```
>You are about to be asked to enter information that will be incorporated 
> into your certificate request.
> What you are about to enter is what is called a Distinguished Name or a DN.
> There are quite a few fields but you can leave some blank
> For some fields there will be a default value,
> If you enter '.', the field will be left blank.
> -----
> Country Name (2 letter code) [XX]:de
> State or Province Name (full name) []:
> Locality Name (eg, city) [Default City]:
> Organization Name (eg, company) [Default Company Ltd]:
> Organizational Unit Name (eg, section) []:
> Common Name (eg, your name or your server's hostname) []:my.proxmox-server.de 
> Email Address []:<YOUR EMAIL!!!!>
> Please enter the following 'extra' attributes
> to be sent with your certificate request
> A challenge password []:<YOUR CHALLENGE-PASS!!!>
> An optional company name []:
>```

***create the self-signed cert using the just created csr:***
```Bash
 # openssl x509 -req -days 365 -in csr.pem -signkey private_key.pem -out certificate.pem
```

>```
>Certificate request self-signature ok
>...
>```
>

***check out the files:***

```Bash 
# ls
```
>```
 > certificate.pem  csr.pem   encrypted_private_key.pem  private_key.pem ...
 >```
 
 ***upload your cert + encrypted privatekey to proxmox:***
 ![upload_ssl](https://github.com/ji-podhead/RHEL_9_Foreman_Guide/blob/main/img/proxmox_upload_custom_certificat.png?raw=true)
 
 - restart proxmox (should happen by default) 

## configure foreman

***configure firewall:***

```Bash
# firewall-cmd --add-port=5900-5930/tcp
# firewall-cmd --add-port=5900-5930/tcp --permanent
```

***install [foreman_fog_proxmox](https://github.com/theforeman/foreman_fog_proxmox):***

```Bash
# sudo dnf install rubygem-foreman_fog_proxmox
```

***restart foreman service:***

```Bash
# sudo systemctl restart foreman.service
```

> - if you get error in foreman-ui after that try this:
> ```Bash
> # foreman-rake db:migrate
> # systemctl restart foreman.service 
>```

***add the proxmox-computeresource:***

- apperently theres seems to be a bug in foreman_fog_proxmox, so we cant use user-token authentication:
![usertoken_bug](https://github.com/ji-podhead/RHEL_9_Foreman_Guide/blob/main/img/proxmox_compute_resource_version.png?raw=true)

-  but at least we dont get the previously mentioned error because of missing ssl cert
- so we switch to access ticket, fill in our proxmox user (needs to be priviliged), as well as our proxmox pasword and finish the compute resource setup:

![finish_compute_resource](https://github.com/ji-podhead/RHEL_9_Foreman_Guide/blob/main/img/proxmox_compute_resource_finish.png?raw=true)

---


| [Knowledge Base](https://ji-podhead.github.io/RHEL_9_Foreman_Guide/knowledge%20base)| [Install](https://ji-podhead.github.io/RHEL_9_Foreman_Guide/installation%20(katello%2Cdiscovery%2Cdhcp%2Ctftp)) | [Discovery and Provisioning](https://ji-podhead.github.io/RHEL_9_Foreman_Guide/discovery%20and%20provisioning) | [libvirt](https://ji-podhead.github.io/RHEL_9_Foreman_Guide/libvirt) | [proxmox](https://ji-podhead.github.io/RHEL_9_Foreman_Guide/proxmox) | [diskless pxe-boot using zfs](https://ji-podhead.github.io/RHEL_9_Foreman_Guide/diskless_pxe_using_zfs) |
