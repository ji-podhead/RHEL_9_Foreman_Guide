





 | [Knowledge Base](https://ji-podhead.github.io/RHEL_9_Foreman_Guide/knowledge%20base)| [Install](https://ji-podhead.github.io/RHEL_9_Foreman_Guide/installation%20(katello%2Cdiscovery%2Cdhcp%2Ctftp)) | [Discovery and Provisioning](https://ji-podhead.github.io/RHEL_9_Foreman_Guide/discovery%20and%20provisioning) | [libvirt](https://ji-podhead.github.io/RHEL_9_Foreman_Guide/libvirt) | [proxmox](https://ji-podhead.github.io/RHEL_9_Foreman_Guide/proxmox) | [diskless pxe-boot using zfs](https://ji-podhead.github.io/RHEL_9_Foreman_Guide/diskless_pxe_using_zfs) |


## *Foreman in a nested VM* managing external DNS & DHCP with Dynamic Updates & RNDC
> - we will install & configure a Foreman-machine running inside a `Rocky Linux`-based VM
> - we will install & configure our DHCP & DNS in `a seperate Debian-based VM`
> - we will configure our DHCP to get managed by Foreman and `share its leases`
> - we will configure Foreman to `manage our external DHCP and DNS`
> - this Guide will also cover how to `debug your servers` and monitor the network 
> - in addition the Guide provides a `walk trough the Discovery process`   

---
<table style="border-collapse: collapse; width: 100%;">
    <tr>
        <td style="width: 50%; vertical-align: top;">
            <table>
                <tr>
                    <th colspan="2" style="background-color: #f0f0f0; text-align: center;">Specs used in this Guide</th>
                </tr>
                <tr>
                    <td style="padding: 8px; border: 1px solid #ddd;">Foreman-Machine-OS</td>
                    <td style="padding: 8px; border: 1px solid #ddd;">Rocky Linux 9.4</td>
                        <tr>
		    <tr>
		   <tr>
                    <td style="padding: 8px; border: 1px solid #ddd;">DHCP & DNS-Machine-OS</td>
                    <td style="padding: 8px; border: 1px solid #ddd;">Debian 12</td>
                        <tr>
                    <td style="padding: 8px; border: 1px solid #ddd;">Host</td>
                    <td style="padding: 8px; border: 1px solid #ddd;">192.168.122.1</td>
                        <tr>
                    <td style="padding: 8px; border: 1px solid #ddd;">PXE-Test-Machine</td>
                 <td style="padding: 8px; border: 1px solid #ddd;">192.169.122.138</td>
                         </tr>
                             <tr>
                     <td style="padding: 8px; border: 1px solid #ddd;">Foreman-Proxy</td>
                <td style="padding: 8px; border: 1px solid #ddd;">192.168.122.20 </td>
                        </tr>
                            <tr>
                    <td style="padding: 8px; border: 1px solid #ddd;">Foreman-FQDN</td>
                <td style="padding: 8px; border: 1px solid #ddd;">foreman.de </td>
                        <tr>
                    <td style="padding: 8px; border: 1px solid #ddd;">DNS & THCP-Server</td>
                <td style="padding: 8px; border: 1px solid #ddd;"> 192.168.122.7</td>
                    </tr>
                </tr>
		    <th colspan="2" style="background-color: #f0f0f0; text-align: center;">Subnet</th>
			    <tr><td>adress<td>192.168.122.0</td></tr> 
			    <tr><td>range</td><td>192.168.122.1 192.168.122.254</td></tr>
			    <tr><td>option routers</td><td> 192.168.122.7</td></tr> 
			    <tr><td>option broadcast-address</td><td>192.168.122.255</td></tr>
			    </td>
                </tr>
            </table>
        </td>
        <td style="width: 50%; vertical-align: top;">
            <h3>Preview</h3>
            <img src="https://github.com/ji-podhead/RHEL_9_Foreman_Guide/blob/main/docs/nestedVM_with_external_DHCP&DNS/images/final_discovery_nestedv_flowchart.png?raw=true" align="center" width="500" />
        </td>
    </tr>
</table>


---

 ***Please proceed with the DNS section of my [DNS-Network Guide](https://ji-podhead.github.io/Network-Guides/DNS/install/) if needed:***
 - All DNS-related topics needed are explained in detail here:
> - [Knowledge Base ](https://ji-podhead.github.io/Network-Guides/DNS/Knowledge%20Base)
> - [Install & Config](https://ji-podhead.github.io/Network-Guides/DNS/install)
> - [Test & Debug](https://ji-podhead.github.io/Network-Guides/DNS/testAndDebug)
> - [Attack- Vectors and Scenario](https://ji-podhead.github.io/Network-Guides/DNS/attackVectorsAndScenario)
> - [Security & Protection](https://ji-podhead.github.io/Network-Guides/DNS/protection)


---

###  DHCP & DNS installation & configuration steps
- create a seperate `debian-based` machine 
- setup your `Bind9 DNS` and `ISC-DHCP`
	- I coulnd get my DHCP on my Foreman Machine to work with the provided Proxmox-NIC
- we create a `RNDC-key` and set up `dynamic updates` in our DHCP and DNS
- **Foreman wont register your machines, even if they have a valid tftp connection, unless you share the leases of DHCP!** 
> otherwise you will get this error in the proxy logs: 
>```json
>Started POST /api/v2/discovered_hosts/facts
>Finished POST /api/v2/discovered_hosts/facts with 404 (1.07 ms) 
>```
> and the discovery image will post a  `404` as well:
>
> <img src="https://github.com/ji-podhead/RHEL_9_Foreman_Guide/blob/main/docs/nestedVM_with_external_DHCP&DNS/images/foreman_nestedVM_failed.png?raw=true" align="center" height="200" />

- ***Therefore these procedures have to get accomplished:***
	- 1.  [Configuring an external DHCP server to use with Foreman server](https://docs.theforeman.org/nightly/Installing_Server/index-foreman-deb.html#configuring-an-external-dhcp-server_foreman)
    
	- 2.  [Configuring Foreman server with an external DHCP server](https://docs.theforeman.org/nightly/Installing_Server/index-foreman-deb.html#Configuring_Server_with_an_External_DHCP_Server_foreman)
> - *both procedures will be covered in this  guide*
- I was to lazy and directly installed the external servers on my Proxmox-Machine, which is stupid:
	- DNS holds a huge risk when misconfigured or attacked
	- if your DNS starves, it will also starve all your Proxmox-stuff and might even damage the Filesystem 



---


### Setup a test machine for pxe boot

- make sure to add your NIC to Boot-Options

  <img src="https://github.com/ji-podhead/RHEL_9_Foreman_Guide/blob/main/docs/nestedVM_with_external_DHCP&DNS/images/foreman_nestedVM_pxeboot.png?raw=true" align="center" height="200" />

  ---

## DHCP & DNS configs for Dynamic Updates & RNDC
 - create a RNDC key
  ```Bash
   #  echo rndc-confgen >> /etc/bind/rndc.conf
   #  chmod 660 /etc/bind/rndc.conf
   #  chown root:bind /etc/bind/rndc.conf
  ```
---

### edit your configs accordingly:

***named.conf***
> `/etc/bind/named.conf`
```yaml
include "/etc/bind/named.conf.options";
include "/etc/bind/named.conf.local";
include "/etc/bind/named.conf.default-zones";
```
---

***named.conf.local***
> `/etc/bind/named.conf.local`
```yaml
include "/etc/bind/rndc.conf";
controls {
  inet 127.0.0.1 port 953 allow {
    127.0.0.1;
    192.168.122.1;
  } keys { "rndc-key"; }; #We can now refer to the key with this variable
};

zone "foreman.de" IN {
        type master;
        file "/etc/bind/zones/foreman.de";
         allow-query { any; };  
        allow-update { key rndc-key; };
};
zone "122.168.192.in-addr.arpa" IN {
        type master;
        file "/etc/bind/zones/foreman.de.rev";
         allow-query { any; };
        allow-update { key rndc-key; };
};
```
---

***named.conf.options***
> `/etc/bind/named.conf.options`
```yaml
acl internals { 127.0.0.0/8; 192.168.122.0/24; };
controls { inet 127.0.0.1 port 953 allow { 127.0.0.1; }; };
options {
  directory "/var/cache/bind";
  forwarders { 192.168.2.1; };
  allow-query { internals; }; # only interal allowed to do queries 
  dnssec-validation auto;
  auth-nxdomain no;    # conform to RFC1035
  listen-on-v6 { none; };
  listen-on { 127.0.0.1; 192.168.122.7; };
  # recursion no;  <--- we allow recursion only for internals for security reason
  allow-recursion { internals; };
  querylog yes; # Enable for debugging
  version "not available"; # Disable for security
};
```

---


***forward-lookup-zone `foreman.de`***
> `/etc/bind/zones/foreman.de`
```yaml
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     foreman.de. root.foreman.de. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      bindserver.foreman.de.
; A record for name server
bindserver      IN      A       192.168.122.20
@       IN      NS      localhost.
@       IN      A       192.168.122.20
@       IN      AAAA    ::1
;ns.foreman.de.    IN    A    192.168.122.7
;ns2.foreman.de.   IN    A    192.168.122.8
```

---

***reverse-lookup-zone `foreman.de.rev`***
> `/etc/bind/zones/foreman.de.rev`
```yaml
; BIND reverse data file for local loopback interface
;
$TTL    604800
@       IN      SOA     foreman.de. root.foreman.de. (
                              1         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
; Name server record
@       IN      NS     bindserver.foreman.de.
; A record for name server
bindserver      IN      A      192.168.122.20
;20.122.168.192.in-addr.arpa. IN PTR foreman.de. <<- this was allready declared in our zone, so we use the syntax below
20      IN      PTR     foreman.de.
@       IN      NS      localhost.
1.0.0   IN      PTR     localhost.
```

---

***dhcp.conf***
> `/etc/dhcp/dhcpd.conf`
```yaml
update-static-leases on;
use-host-decl-names on;
option domain-name "foreman.de.";

# Path to the key for dynamic updates
include "/etc/bind/rndc.key";

# Deactivating optimization and checking conflicts of dynamic updates
update-optimization off;
update-conflict-detection off;

# Embedding the configoration of Remote Control
include "/etc/bind/rndc.conf";

# Definition der Zone für Ihren Domainnamen
zone foreman.de. {
        primary 192.168.122.7; # DNS-Server-IP
        key rndc-key;
}

# Definition of the Reverse Lookup Zone
zone foreman.de. {
        primary 192.168.122.7; # DNS-Server-IP
        key rndc-key;
}

# Subnet-Konfiguration
subnet 192.168.122.0 netmask 255.255.255.0 {
  range 192.168.122.1 192.168.122.254;
  option subnet-mask 255.255.255.0;
  option routers 192.168.122.20;
  option broadcast-address 192.168.122.255;
  dynamic-update;
  option domain-name "foreman.de";
  option domain-name-servers 192.168.122.7;
}

##################################################################################
#		THIS WILL BE REQUIRED BY FOREMAN LATER
##################################################################################
#		---------------------------------------------------
# 		>> `ssec-keygen -a DH -b 512 -n HOST omapi_key` <<
#		---------------------------------------------------
# 		- copy the private key and paste it here
#		- we choose DUFFIE HILBERT encryption here 
#		- i coulndt get dnssec to generate hmac-sha256 encryption keys
# 		- you can use TSIG to gen. hmac-sha256 encryption keys though:
# 				- tsig-keygen >> omapi.key
#		- but instead i used dnssec and checked --help for available algos
#		---------------------------------------------------
# omapi-port 7911;
# key omapi_key {
#        algorithm DH;
#        secret # "Rf8oLo11/SYUi0ulXc+EAt9meiZPXOA0QqJR779UDV0xRphg0jwU55yapEViRqytMn0gy7ohtytZrVa6UzJkjQ==";
# };
# omapi-key omapi_key;
#########################################################################
```

---

***Always make sure to update Bind9 when changing configs!!!***

**edit AppArmor** 
> - <u>*if you fail to restart isc-dhcp*</u>
 
```Bash
# sudo nano /etc/apparmor.d/usr.sbin.dhcpd  
```

> add 
> ```perl
>/etc/bind/ rw,
>/etc/bind/** rw,
>```

restart AppArmor:

```Bash
# apparmor_parser -r /etc/apparmor.d/usr.sbin.dhcpd  
```

  **restart/refresh DNS & DHCP**
  
```Bash
# named-checkzone foreman.de /etc/bind/zones/foreman.de
# named-checkzone foreman.de /etc/bind/zones/foreman.de.rev
# named-checkconf /etc/bind/named.conf.options
# named-checkconf
# sudo systemctl restart bind9
# sudo systemctl restart isc-dhcp-server
```  
´

---

## Initialize Foreman with Discovery Plugin
- get the repos, configure firewall...etc
	- everything you need to know is explained in detail in the [install section of this guide](https://ji-podhead.github.io/RHEL_9_Foreman_Guide/installation%20(katello%2Cdiscovery%2Cdhcp%2Ctftp))
> -  <u>*but dont upgrade foreman to use managed DNS & DHCP  yet!!*</u>
> - ***set managed DNS & DHCP to false:***
>```Bash
>foreman-installer \ 
>--foreman-proxy-dns true \
>--foreman-proxy-dns-managed false \ 
>--foreman-proxy-dhcp true \
>--foreman-proxy-dhcp-managed false
>--foreman-proxy-tftp true \
>--foreman-proxy-tftp-managed true \
>--foreman-proxy-tftp-servername 192.168.122.20
>```

---

***check if Discovery Plugin created the boot image files***

  <img src="https://github.com/ji-podhead/RHEL_9_Foreman_Guide/blob/main/docs/nestedVM_with_external_DHCP&DNS/images/foreman_nestedVM_missing_bootFolder.png?raw=true" align="center" height="200" />

- in my case they where missing, so i had to copy the image intoo the boot folder
	 - i used my nfs, but you can of course use ***securecopy*** as well  
- the outcome is that the pxe boot loader constantly tries to boot (repeats the counter on the blue screen)
	- ***this typically means that theres a TFTP-misconfiguration***
---

***configure `pxelinux.cfg/default`***

```yaml
LABEL discovery
  MENU LABEL Foreman Discovery Image
  KERNEL boot/fdi-image/vmlinuz0
  APPEND initrd=boot/fdi-image/initrd0.img rootflags=loop root=live:/fdi.iso rootfstype=auto ro rd.live.image  roxy.url=https://foreman.de proxy.type=foreman
  IPAPPEND 2
  ```

---

***configure Foreman to be ready for discovery & provisioning***
- add a subnet, as well as a hostgroup and configure foreman 
- everything you need to know is explained in detail in the [discovery & provisioning section](https://ji-podhead.github.io/RHEL_9_Foreman_Guide/discovery%20and%20provisioning)

---
<u>*we will not upgrade Foreman to manage DNS & DHCP yet!*</u>
> -  first we need to configure our DNS & DHCP, as well as Foreman to manage our external servers,  which we will do int the next step 

---



## Configure DHCP & NFS to share the leases

 install isc-dhcp-server

```Bash
# apt install isc-dhcp-server -y
```

 configure Firewall (debian)

```Bash
# sudo apt-get install iptables-persistent netfilter-persistent
# sudo iptables -A INPUT -p tcp --dport 7911 -j 
# sudo iptables -A INPUT -p tcp --syn --dport 2049 -j 
# sudo iptables -A INPUT -p udp --dport 2049 -j 
# sudo iptables -A INPUT -p tcp --dport 111 -j 
# sudo iptables -A INPUT -p udp --dport 111 -j 
# sudo iptables -A INPUT -p tcp --dport 32765:61000 -j 
# sudo iptables -A INPUT -p udp --dport 32765:61000 -j 
# sudo netfilter-persistent save
# sudo iptables-save > /etc/iptables/rules.v4
# sudo netfilter-persistent reload
```
---

***add the Foreman user***

```Bash
  #  useradd -u 982 -g 982 -s /sbin/nologin foreman
  #  sudo usermod -u 982 -g 982 foreman 
  ```
> user and group can be found out via foreman-machine like this:
>```
># id -u foreman
># id -g foreman
>```

   restore the read and execute flags:
```Bash
    # chmod o+rx /etc/dhcp/
    # chmod o+r /etc/dhcp/dhcpd.conf
    # chattr +i /etc/dhcp/ /etc/dhcp/dhcpd.conf
```

---


***setup nfs***

install nfs and  create the export paths

```Bash
  #  sudo apt-get install nfs-kernel-server
  #  systemctl enable --now nfs-server
  #  mkdir -p /exports/var/lib/dhcpd /exports/etc/dhcp
```

 start the nfs server
```Bash
 # systemctl enable --now nfs-server
 ```

> - you can check the status like this:
> ```bash
> # sudo systemctl status nfs-kernel-server
>```

 edit the fstab for persistent nfs export

```Bash
# nano /etc/fstab
```

>```yaml
>/var/lib/dhcp /exports/var/lib/dhcpd none bind,auto 0 0
>/etc/dhcp /exports/etc/dhcp none bind,auto 0 0
>```

 reload the Daemon and mount everything in fstab using `mount -a`

```Bash
# systemctl daemon-reload
# mount -a
```
 edit the exports file and activate our nfs

```Bash
# nano /etc/exports
# exportfs -rva
```

> the exports file should look like this:
> ```yaml
>/exports 192.168.192.20(rw,async,no_root_squash,fsid=0,no_subtree_check)
>/exports/etc/dhcp 192.168.122.20(ro,async,no_root_squash,no_subtree_check,nohide)
>/exports/var/lib/dhcpd 192.168.122.20(ro,async,no_root_squash,no_subtree_check,nohide)

---

***omapi-key***

```Bash
# cd /etc/bind
# ssec-keygen -a DH -b 512 -n HOST omapi_key
# ls
```

> alternatively you can use TSIG for hmac-sha256 encryption keys
>```
># tsig-keygen >> omapi.key
>```


- print the generated file: `002+57454.private`

```
 cat Komapi_key.+002+57454.private
```


- copy the private key and paste it in the `omapi-key definition` of your `dhcpd.conf`
>```
>omapi-port 7911;
>key omapi_key {
 >       algorithm DH;
 >       secret >"Rf8oLo11/SYUi0ulXc+EAt9meiZPXOA0QqJR779UDV0xRphg0jwU55yapEViRqytMn0gy7ohtytZrVa6UzJkjQ==";
>};
>omapi-key omapi_key;
>```

---

 start the dhcp server

```bash
# systemctl enable --now dhcpd
```


---


## Configure Foreman for external DNS & DHCP management




Follow the procedure described in the [Foreman Doc's](https://docs.theforeman.org/nightly/Installing_Server/index-foreman-deb.html#Configuring_Server_with_an_External_DHCP_Server_foreman)

```bash
# sudo dnf install nfs-common
# mkdir -p /mnt/nfs/etc/dhcp /mnt/nfs/var/lib/dhcpd
# chown -R foreman-proxy /mnt/nfs
# showmount -e foreman.de
# rpcinfo -p foreman.de
```

edit `/etc/fstab`

>```
>192.168.122.7:/exports/etc/dhcp /mnt/nfs/etc/dhcp
>ro,vers=3,auto,nosharecache,context="system_u:object_r:dhcp_etc_t:s0" 0 0
>
>192.168.122.7:/exports/var/lib/dhcpd /mnt/nfs/var/lib/dhcpd
>ro,vers=3,auto,nosharecache,context="system_u:object_r:dhcpd_state_t:s0" 0 0
>```

reload the daemon and mount:

```bash
# systemctl daemon-reload
# mount -a
```
> dont worry if you get a warining like this if you have additional stuff in the fstab like root fs:
> - `mount: 0: der Einhängepunkt ist nicht vorhanden.`

---

if it fails you can debug like this:
```bash
# mount -t nfs 192.168.122.7:/exports/etc/dhcp /mnt/nfs/etc/dhcp 
# cd /mnt/nfs/etc/dhcp
# ls
```
>```
>debug          dhclient-enter-hooks.d  dhcpd6.conf  Komapi_key.+002+57454.key      old.conf   rndc.conf
>dhclient.conf  dhclient-exit-hooks.d   dhcpd.conf   Komapi_key.+002+57454.private  omapi.key  rndc.key
>```
>
> - you can also try to disable firewall
>```
> # sudo sytsemctl disable firewalld
>```
>

---

***check the shared leases and dhcpd.conf***

```bash
# ls /mnt/nfs/var/lib/dhcpd
```
>```
>dhcpd6.leases  dhcpd6.leases~  dhcpd.leases  dhcpd.leases~
>```

login to foreman-proxy user
```bash
#  su foreman-proxy -s /bin/bash
```
check if the user has access to the files

```bash
bash-5.1$ cat /mnt/nfs/etc/dhcp/dhcpd.conf
```

>```
>authoritative;
>default-lease-time 14400;
>max-lease-time 18000;
>log-facility local7;
>...
>```

```bash
bash-5.1$  cat /mnt/nfs/var/lib/dhcpd/dhcpd.leases
```

>```
># The format of this file is documented in the dhcpd.leases(5) manual page.
># This lease file was written by isc-dhcp-4.4.3-P1
>...
>```

---

***external DNS config***
follow the procedure  from the [API Doc’s](https://docs.theforeman.org/nightly/Installing_Server/index-foreman-deb.html#configuring-external-services)
copy the `rndc.key` from the external machine to the foreman machine
```Bash
   nano /etc/rndc.key
```
>```
>key "rndc-key" {
>        algorithm hmac-sha256;
 >       secret "VsU3++3blrsWTODlA2AzToXebHMOa96ysmWzq3Q0LiA=";
>};
>```

```bash
   #  restorecon -v /etc/rndc.key
   #  chown -v root:named /etc/rndc.key
   #  chmod -v 640 /etc/rndc.key
   #  usermod -a -G named foreman-proxy
```
---

## Upgrade Foreman




***external DNS&DHCP-Management Upgrade***

```Bash
  foreman-installer \ 
  --foreman-proxy-dns true \
  --foreman-proxy-dns-managed true \
  --foreman-proxy-dns-provider=nsupdate \
  --foreman-proxy-dns-ttl=86400 \
  --foreman-proxy-keyfile=/etc/rndc.key \
  --foreman-proxy-dhcp true \
  --foreman-proxy-dhcp-managed true \
  --foreman-proxy-dhcp-range "192.168.122.1 192.168.122.254" \
  --foreman-proxy-dhcp-gateway 192.168.122.7 \
  --foreman-proxy-dhcp-nameservers 192.168.122.7 \
  --foreman-proxy-tftp true \
  --foreman-proxy-tftp-managed true \
  --foreman-proxy-tftp-servername 192.168.122.20
```

---


***remote-isc-key Upgrade***
```
foreman-installer \
--enable-foreman-proxy-plugin-dhcp-remote-isc \
--foreman-proxy-dhcp-provider=remote_isc \
--foreman-proxy-dhcp-server=foreman.de \
--foreman-proxy-dhcp=true \
--foreman-proxy-plugin-dhcp-remote-isc-dhcp-config /mnt/nfs/etc/dhcp/dhcpd.conf \
--foreman-proxy-plugin-dhcp-remote-isc-dhcp-leases /mnt/nfs/var/lib/dhcpd/dhcpd.leases \
--foreman-proxy-plugin-dhcp-remote-isc-key-name=omapi_key \
--foreman-proxy-plugin-dhcp-remote-isc-key-secret=Rf8oLo11/SYUi0ulXc+EAt9meiZPXOA0QqJR779UDV0xRphg0jwU55yapEViRqytMn0gy7ohtytZrVa6UzJkjQ== \
--foreman-proxy-plugin-dhcp-remote-isc-omapi-port=7911
```

restart foreman-proxy just in case

```Bash
sudo systemctl restart foreman-proxy
```
---

## Discovery walktrough and debugging
   <img src="https://github.com/ji-podhead/RHEL_9_Foreman_Guide/blob/main/docs/nestedVM_with_external_DHCP&DNS/images/final_discovery_nestedv_flowchart.png?raw=true" align="center" />

## Successfull Discovery Logs

***tail /var/logs/foreman/production.log -f***
```
2024-06-09T20:49:03 [I|app|8832e8d0] Started GET "/notification_recipients" for 192.168.122.1 at 2024-06-09 20:49:03 +0200
2024-06-09T20:49:03 [I|app|8832e8d0] Processing by NotificationRecipientsController#index as JSON
2024-06-09T20:49:03 [I|app|8832e8d0] Completed 200 OK in 11ms (Views: 0.2ms | ActiveRecord: 2.5ms | Allocations: 2231)
2024-06-09T20:49:13 [I|app|cdbc9b4d] Started GET "/notification_recipients" for 192.168.122.1 at 2024-06-09 20:49:13 +0200
2024-06-09T20:49:13 [I|app|cdbc9b4d] Processing by NotificationRecipientsController#index as JSON
2024-06-09T20:49:13 [I|app|cdbc9b4d] Completed 200 OK in 11ms (Views: 0.2ms | ActiveRecord: 2.3ms | Allocations: 2231)
2024-06-09T20:49:23 [I|app|432f5b59] Started GET "/notification_recipients" for 192.168.122.1 at 2024-06-09 20:49:23 +0200
2024-06-09T20:49:23 [I|app|432f5b59] Processing by NotificationRecipientsController#index as JSON
2024-06-09T20:49:23 [I|app|432f5b59] Completed 200 OK in 11ms (Views: 0.1ms | ActiveRecord: 2.3ms | Allocations: 2231)
2024-06-09T20:49:30 [I|app|767c5aea] Started POST "/api/v2/discovered_hosts/facts" for 192.168.122.138 at 2024-06-09 20:49:30 +0200
2024-06-09T20:49:30 [I|app|767c5aea] Processing by Api::V2::DiscoveredHostsController#facts as JSON
2024-06-09T20:49:30 [I|app|767c5aea]   Parameters: {"facts"=>"[FILTERED]", "apiv"=>"v2", "discovered_host"=>{"facts"=>"[FILTERED]"}}
2024-06-09T20:49:30 [I|aud|767c5aea] Nic::Managed (2) create event on mac 
2024-06-09T20:49:30 [I|aud|767c5aea] Nic::Managed (2) create event on ip 
2024-06-09T20:49:30 [I|aud|767c5aea] Nic::Managed (2) create event on type Nic::Managed
2024-06-09T20:49:30 [I|aud|767c5aea] Nic::Managed (2) create event on name mac525400355ead
2024-06-09T20:49:30 [I|aud|767c5aea] Nic::Managed (2) create event on host_id 2
2024-06-09T20:49:30 [I|aud|767c5aea] Nic::Managed (2) create event on subnet_id 
2024-06-09T20:49:30 [I|aud|767c5aea] Nic::Managed (2) create event on domain_id 
2024-06-09T20:49:30 [I|aud|767c5aea] Nic::Managed (2) create event on attrs {}
2024-06-09T20:49:30 [I|aud|767c5aea] Nic::Managed (2) create event on provider 
2024-06-09T20:49:30 [I|aud|767c5aea] Nic::Managed (2) create event on username 
2024-06-09T20:49:30 [I|aud|767c5aea] Nic::Managed (2) create event on password [redacted]
2024-06-09T20:49:30 [I|aud|767c5aea] Nic::Managed (2) create event on virtual false
2024-06-09T20:49:30 [I|aud|767c5aea] Nic::Managed (2) create event on link true
2024-06-09T20:49:30 [I|aud|767c5aea] Nic::Managed (2) create event on identifier 
2024-06-09T20:49:30 [I|aud|767c5aea] Nic::Managed (2) create event on tag 
2024-06-09T20:49:30 [I|aud|767c5aea] Nic::Managed (2) create event on attached_to 
2024-06-09T20:49:30 [I|aud|767c5aea] Nic::Managed (2) create event on managed true
2024-06-09T20:49:30 [I|aud|767c5aea] Nic::Managed (2) create event on mode balance-rr
2024-06-09T20:49:30 [I|aud|767c5aea] Nic::Managed (2) create event on attached_devices 
2024-06-09T20:49:30 [I|aud|767c5aea] Nic::Managed (2) create event on bond_options 
2024-06-09T20:49:30 [I|aud|767c5aea] Nic::Managed (2) create event on primary true
2024-06-09T20:49:30 [I|aud|767c5aea] Nic::Managed (2) create event on provision true
2024-06-09T20:49:30 [I|aud|767c5aea] Nic::Managed (2) create event on compute_attributes {}
2024-06-09T20:49:30 [I|aud|767c5aea] Nic::Managed (2) create event on execution false
2024-06-09T20:49:30 [I|aud|767c5aea] Nic::Managed (2) create event on ip6 
2024-06-09T20:49:30 [I|aud|767c5aea] Nic::Managed (2) create event on subnet6_id 
2024-06-09T20:49:30 [I|aud|767c5aea] Host::Base (2) create event on name mac525400355ead
2024-06-09T20:49:30 [I|aud|767c5aea] Host::Base (2) create event on last_compile 
2024-06-09T20:49:30 [I|aud|767c5aea] Host::Base (2) create event on root_pass 
2024-06-09T20:49:30 [I|aud|767c5aea] Host::Base (2) create event on architecture_id 
2024-06-09T20:49:30 [I|aud|767c5aea] Host::Base (2) create event on operatingsystem_id 
2024-06-09T20:49:30 [I|aud|767c5aea] Host::Base (2) create event on ptable_id 
2024-06-09T20:49:30 [I|aud|767c5aea] Host::Base (2) create event on medium_id 
2024-06-09T20:49:30 [I|aud|767c5aea] Host::Base (2) create event on build false
2024-06-09T20:49:30 [I|aud|767c5aea] Host::Base (2) create event on comment 
2024-06-09T20:49:30 [I|aud|767c5aea] Host::Base (2) create event on disk 
2024-06-09T20:49:30 [I|aud|767c5aea] Host::Base (2) create event on installed_at 
2024-06-09T20:49:30 [I|aud|767c5aea] Host::Base (2) create event on model_id 
2024-06-09T20:49:30 [I|aud|767c5aea] Host::Base (2) create event on hostgroup_id 
2024-06-09T20:49:30 [I|aud|767c5aea] Host::Base (2) create event on owner_id 
2024-06-09T20:49:30 [I|aud|767c5aea] Host::Base (2) create event on owner_type 
2024-06-09T20:49:30 [I|aud|767c5aea] Host::Base (2) create event on enabled true
2024-06-09T20:49:30 [I|aud|767c5aea] Host::Base (2) create event on puppet_ca_proxy_id 
2024-06-09T20:49:30 [I|aud|767c5aea] Host::Base (2) create event on managed false
2024-06-09T20:49:30 [I|aud|767c5aea] Host::Base (2) create event on use_image 
2024-06-09T20:49:30 [I|aud|767c5aea] Host::Base (2) create event on image_file 
2024-06-09T20:49:30 [I|aud|767c5aea] Host::Base (2) create event on uuid 
2024-06-09T20:49:30 [I|aud|767c5aea] Host::Base (2) create event on compute_resource_id 
2024-06-09T20:49:30 [I|aud|767c5aea] Host::Base (2) create event on puppet_proxy_id 
2024-06-09T20:49:30 [I|aud|767c5aea] Host::Base (2) create event on certname 
2024-06-09T20:49:30 [I|aud|767c5aea] Host::Base (2) create event on image_id 
2024-06-09T20:49:30 [I|aud|767c5aea] Host::Base (2) create event on organization_id 
2024-06-09T20:49:30 [I|aud|767c5aea] Host::Base (2) create event on location_id 
2024-06-09T20:49:30 [I|aud|767c5aea] Host::Base (2) create event on otp 
2024-06-09T20:49:30 [I|aud|767c5aea] Host::Base (2) create event on realm_id 
2024-06-09T20:49:30 [I|aud|767c5aea] Host::Base (2) create event on compute_profile_id 
2024-06-09T20:49:30 [I|aud|767c5aea] Host::Base (2) create event on provision_method 
2024-06-09T20:49:30 [I|aud|767c5aea] Host::Base (2) create event on grub_pass 
2024-06-09T20:49:30 [I|aud|767c5aea] Host::Base (2) create event on global_status 0
2024-06-09T20:49:30 [I|aud|767c5aea] Host::Base (2) create event on lookup_value_matcher 
2024-06-09T20:49:30 [I|aud|767c5aea] Host::Base (2) create event on pxe_loader 
2024-06-09T20:49:30 [I|aud|767c5aea] Host::Base (2) create event on initiated_at 
2024-06-09T20:49:30 [I|aud|767c5aea] Host::Base (2) create event on build_errors 
2024-06-09T20:49:30 [I|aud|767c5aea] Host::Base (2) create event on creator_id 1
2024-06-09T20:49:30 [I|aud|767c5aea] Host::Base (2) create event on discovery_rule_id 
2024-06-09T20:49:33 [I|app|c1b54672] Started GET "/notification_recipients" for 192.168.122.1 at 2024-06-09 20:49:33 +0200
2024-06-09T20:49:34 [I|app|c1b54672] Processing by NotificationRecipientsController#index as JSON
2024-06-09T20:49:34 [I|app|c1b54672] Completed 200 OK in 28ms (Views: 0.2ms | ActiveRecord: 9.3ms | Allocations: 5349)
2024-06-09T20:49:35 [I|app|767c5aea] Import facts for 'mac525400355ead' completed. Added: 279, Updated: 0, Deleted 0 facts
2024-06-09T20:49:35 [I|aud|767c5aea] Model (2) create event on name KVM
2024-06-09T20:49:35 [I|aud|767c5aea] Model (2) create event on info 
2024-06-09T20:49:35 [I|aud|767c5aea] Model (2) create event on vendor_class 
2024-06-09T20:49:35 [I|aud|767c5aea] Model (2) create event on hardware_model 
2024-06-09T20:49:35 [I|aud|767c5aea] Nic::Managed (2) update event on mac , 52:54:00:35:5e:ad
2024-06-09T20:49:35 [I|aud|767c5aea] Nic::Managed (2) update event on identifier , enp1s0
2024-06-09T20:49:35 [I|app|767c5aea] Detected IPv4 subnet: s1 with taxonomy ["Default Organization"]/["Default Location"]
2024-06-09T20:49:35 [I|app|767c5aea] Assigned location: Default Location
2024-06-09T20:49:35 [I|app|767c5aea] Assigned organization: Default Organization
2024-06-09T20:49:35 [I|aud|767c5aea] Host::Base (2) update event on model_id , 2
2024-06-09T20:49:35 [I|aud|767c5aea] Host::Base (2) update event on owner_id , 1
2024-06-09T20:49:35 [I|aud|767c5aea] Host::Base (2) update event on owner_type , User
2024-06-09T20:49:35 [I|aud|767c5aea] Host::Base (2) update event on organization_id , 1
2024-06-09T20:49:35 [I|aud|767c5aea] Host::Base (2) update event on location_id , 2
2024-06-09T20:49:35 [I|aud|767c5aea] Nic::Managed (2) update event on subnet_id , 1
2024-06-09T20:49:35 [I|app|767c5aea] Completed 201 Created in 5468ms (Views: 1.9ms | ActiveRecord: 2906.4ms | Allocations: 548824)
```

