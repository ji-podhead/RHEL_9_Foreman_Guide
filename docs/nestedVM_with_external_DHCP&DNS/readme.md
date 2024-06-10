



 | [Knowledge Base](https://ji-podhead.github.io/RHEL_9_Foreman_Guide/knowledge%20base)| [Install](https://ji-podhead.github.io/RHEL_9_Foreman_Guide/installation%20(katello%2Cdiscovery%2Cdhcp%2Ctftp)) | [Discovery and Provisioning](https://ji-podhead.github.io/RHEL_9_Foreman_Guide/discovery%20and%20provisioning) | [libvirt](https://ji-podhead.github.io/RHEL_9_Foreman_Guide/libvirt) | [proxmox](https://ji-podhead.github.io/RHEL_9_Foreman_Guide/proxmox) | [diskless pxe-boot using zfs](https://ji-podhead.github.io/RHEL_9_Foreman_Guide/diskless_pxe_using_zfs) |


## *Foreman in a nested VM* managing external DNS & DHCP with Dynamic Updates 
> - we will install & configure a Foreman-machine running inside a `Rocky Linux`-based VM
> - we will install & configure our DHCP & DNS in `a seperate Debian-based VM`
> - we will configure our DHCP to get managed by Foreman and share its leases
> - we will configure Foreman to `manage our external DHCP and DNS`
> - this Guide will also cover how to `debug your servers` and monitor the network 
> - in addition the Guide provides a `walktrough trough the Discovery process`   

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

###  DHCP & DNS installation & configuration steps
- create a seperate `debian-based` machine 
- setup your `Bind9 DNS` and `ISC-DHCP`
	- I coulnd get my DHCP on my Foreman Machine to work with the provided Proxmox-NIC
- **Foreman wont register your machines, even if they have a valid tftp connection, unless you share the leases of DHCP!** 
> otherwise you will get this error in the proxy logs: 
>```json
>Started POST /api/v2/discovered_hosts/facts
>Finished POST /api/v2/discovered_hosts/facts with 404 (1.07 ms) 
>```
> and the discovery image will post a  `404` as well:
>
> <img src="https://github.com/ji-podhead/RHEL_9_Foreman_Guide/blob/main/docs/nestedVM_with_external_DHCP&DNS/images/foreman_nestedVM_failed.png?raw=true" align="center" height="200" />

- Therefore these procedures have to get accomplished:
	- 1.  [Configuring an external DHCP server to use with Foreman server](https://docs.theforeman.org/nightly/Installing_Server/index-foreman-deb.html#configuring-an-external-dhcp-server_foreman)
    
	- 2.  [Configuring Foreman server with an external DHCP server](https://docs.theforeman.org/nightly/Installing_Server/index-foreman-deb.html#Configuring_Server_with_an_External_DHCP_Server_foreman)
- both procedures will be covered in this  guide
- I was to lazy and directly installed on my Proxmox-Machine, which is stupid:
	- DNS holds a huge risk when misconfigured or attacked
	- if your DNS starves, it will also starve all your Proxmox-stuff and might even damage the Filesystem 

---

 ***Please proceed with the DNS section of my [DNS-Network Guide](https://ji-podhead.github.io/Network-Guides/DNS/install/) if needed:***
 - All DNS-related topics needed are explained in detail here:
> - [Knowledge Base ](https://ji-podhead.github.io/Network-Guides/DNS/Knowledge%20Base)
> - [Install & Config](https://ji-podhead.github.io/Network-Guides/DNS/install)
> - [Test & Debug](https://ji-podhead.github.io/Network-Guides/DNS/testAndDebug)
> - [Attack- Vectors and Scenario](https://ji-podhead.github.io/Network-Guides/DNS/attackVectorsAndScenario)
> - [Security & Protection](https://ji-podhead.github.io/Network-Guides/DNS/protection)


---


### Setup a test machine for pxe boot

- make sure to add your NIC to Boot-Options

  <img src="https://github.com/ji-podhead/RHEL_9_Foreman_Guide/blob/main/docs/nestedVM_with_external_DHCP&DNS/images/foreman_nestedVM_pxeboot.png?raw=true" align="center" height="200" />

  ---

## Dynamic Updates & Shared Leases
 - create a rdnc key
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

########################################################################
#		THIS WILL BE REQUIRED BY FOREMAN LATER
#			- we choose DUFFIE HILBERT encryption here
#			- i coulndt get dnssec to generate encryption keys
#			- instead i used TSIG and checked help for available algos
########################################################################
omapi-port 7911;
key omapi_key {
        algorithm DH;
        secret "Rf8oLo11/SYUi0ulXc+EAt9meiZPXOA0QqJR779UDV0xRphg0jwU55yapEViRqytMn0gy7ohtytZrVa6UzJkjQ==";
};
omapi-key omapi_key;
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
- everything you need to know is explained in detail in the [discovery & provisioning section of this guide](https://ji-podhead.github.io/RHEL_9_Foreman_Guide/discovery%20and%20provisioning)

---
<u>*we will not upgrade Foreman to manage DNS & DHCP yet!*</u>
> -  first we need to configure our DNS & DHCP, as well as Foreman to manage our external servers,  which we will do int the next step 

---



## Configure DHCP
- configure Firewall (debian)
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
- edit the fstab for persistent nfs export

```Bash
# nano /etc/fstab
```
>```yaml
>/var/lib/dhcp /exports/var/lib/dhcpd none bind,auto 0 0
>/etc/dhcp /exports/etc/dhcp none bind,auto 0 0
>```
- create the export paths, reload the Daemon and mount everything in fstab using `mount -a`
```Bash
# mkdir -p /exports/var/lib/dhcpd /exports/etc/dhcp
# systemctl daemon-reload
# mount -a
```
- edit the exports file and make changes active

```Bash
# nano /etc/exports
# exportfs -rva
```

> the exports file should look like this:
> ```yaml
>/exports 192.168.192.20(rw,async,no_root_squash,fsid=0,no_subtree_check)
>/exports/etc/dhcp 192.168.122.20(ro,async,no_root_squash,no_subtree_check,nohide)
>/exports/var/lib/dhcpd 192.168.122.20(ro,async,no_root_squash,no_subtree_check,nohide)


***omapi-key***
```Bash
# cd /etc/bind
# tsig-keygen >> omapi.key
# ls
```
> we should see the generated key: `002+57454.private`

```
 cat Komapi_key.+002+57454.private
```
```
  308  nano named.conf.options
  309  history | grep named
  310  named-checkconf /etc/bind/named.conf.local
  311  named-checkconf /etc/bind/named.conf.options
  312  named-checkconf /etc/bind/named.conf.options
  313  named-checkconf
  314  sudo systemctl restart isc-dhcpserver
  315  sudo systemctl restart isc-dhcp-server
  316  journalctl -u named.service -f
  317  dnssec-keygen -a HMAC-MD5 -b 512 -n HOST omapi_key
  318  dnssec-keygen -a HMAC-MD5 -b 512 -n HOST omapi_key
  319  dnssec-keygen -a HMAC-SHA256 -b 512 -n HOST omapi_key
  320   apt-get -y install bind9utils
  321  dnssec-keygen -a HMAC-MD5 -b 512 -n HOST omapi_key
  322  tsig-keygen -a HMAC-MD5 -b 512 -n HOST omapi_key
  323  tsig-keygen -a HMAC-MD5 -n HOST omapi_key
  324  tsig-keygen -a -n HOST omapi_key
  325  tsig-keygen -a  HOST omapi_key
  326  tsig-keygen
  327  cd /etc/dhcp
  328  tsig-keygen >> omapi.key
  329  grep "^Key" omapikey.+*.private | cut -d' ' -f2
  330  grep "^Key" omapi.key.+*.private | cut -d' ' -f2
  331  grep "^Key" omapi.key. | cut -d' ' -f2
  332  grep "^Key" omapi.key | cut -d' ' -f2
  333  ls
  334  cat omapi.key
  335  tsig-keygen >> omapi.key
  336  cat omapi.key
  337  tsig-keygen > omapi.key
  338  cat omapi.key
  339  grep "^Key" omapi.key | cut -d' ' -f2
  340  cat omapi.key
  341  dnssec-keygen  my_tsig_key
  342  dnssec-keygen --help
  343  dnssec-keygen -a DH -b 128 -n HOST omapi_key
  344  dnssec-keygen -a DH -b 512 -n HOST omapi_key
  345  grep ^Key Komapi_key.+*.private | cut -d ' ' -f2
  346  nano /etc/dhcp/dhcpd.conf
  347  ls
  348  cat Komapi_key.+002+57454.private
  349  nano /etc/dhcp/dhcpd.conf
  350  nano /etc/dhcp/dhcpd.conf
  351  firewall-cmd --add-service dhcp
  352  groupadd -g 990 foreman
  353  useradd -u 993 -g 990 -s /sbin/nologin foreman
  354  groupadd -g 982 foreman
  355  sudo groupdel foreman
  356  sudo groupdel foreman
  357  sudo groupmod -g 982 foreman
  358  sudo groupmod -g 982 foreman
  359  useradd -u 982 -g 982 -s /sbin/nologin foreman
  360  sudo usermod -u 982 -g 982 foreman
  361  sudo apt-get install nfs-kernel-server
  362  systemctl enable --now nfs-server
  363  mkdir -p /exports/var/lib/dhcpd /exports/etc/dhcp
  364  /var/lib/dhcpd /exports/var/lib/dhcpd none bind,auto 0 0
  365  nano /etc/fstab
  366  /var/lib/dhcpd /exports/var/lib/dhcpd none bind,auto 0 0
  367  nano /etc/fstab
  368  mount -a
  369  systemctl daemon-reload
  370  mount -a
  371  nano /etc/fstab
  372  mkdir -p /exports/var/lib/dhcpd /exports/etc/dhcp
  373  mount -a
  374  cd /var/lib/d
  375  cd /var/lib
  376  ls
  377  cd /var/lib/dhcp
  378  ls
  379  nano /etc/fstab
  380  dmesg mount
  381  dmesg
  382  mount -a
  383  nano /etc/fstab
  384  mkdir -p /exports/var/lib/dhcpd /exports/etc/dhcp
  385  cd /exports/var/lib
  386  ls
  387  ls -l 
  388  nano /etc/fstab
  389  sudo mount /var/lib/dhcpd /exports/var/lib/dhcpd
  390  ls /var/lib
  391  nano /etc/fstab
  392  mount -a
  393  systemctl daemon-reload
  394  mount -a
  395  nano /etc/exports
  396  nano /etc/exports
  397  exportfs -rva
  398  sudo iptables -A INPUT -p tcp --dport 7911 -j ACCEPT
  399  sudo apt-get update
  400  sudo apt-get install iptables-persistent
  401  sudo iptables -A INPUT -p tcp --syn --dport 2049 -j ACCEPT
  402  sudo iptables -A INPUT -p udp --dport 2049 -j ACCEPT
  403  sudo iptables -A INPUT -p tcp --dport 111 -j ACCEPT
  404  sudo iptables -A INPUT -p udp --dport 111 -j ACCEPT
  405  sudo iptables -A INPUT -p tcp --dport 32765:61000 -j ACCEPT
  406  sudo iptables -A INPUT -p udp --dport 32765:61000 -j ACCEPT
  407  sudo netfilter-persistent save
  408  sudo iptables-save > /etc/iptables/rules.v4
  409  sudo netfilter-persistent reload
  410  sudo iptables-restore < /etc/iptables/rules.v4
  411  systemstctl status nfs
  412  systemstctl status nfs-utils
  413  systemstctl status nfs-daemon
  414  systemstctl status nfs-common
  415  systemstctl status nfsd
  416  sudo systemctl start nfs-kernel-server
  417  sudo systemctl status nfs-kernel-server
  418  journalctl -u nfs-kernel-server
  419  journalctl -u nfs-kernel
  420  journalctl -u nfs
  421  sudo systemctl status nfs-kernel-server
  422  systemctl daemon-reload
  423  exportfs -v
  424  nano /etc/fstab
  425  nano /etc/dhcp/dhcpd.conf
  426  journalctl -u named.service -f
  427  cat /var/lib/dhcp/dhcpd.leases
  428  nano /etc/dhcp/dhcpd.conf
  429  sudo nano /etc/bind/named.conf.options
  430  sudo nano /etc/bind/named.conf
  431  sudo nano /etc/bind/named.conf.local
  432  nano /etc/bind/zones/foreman.de
  433  nano /etc/bind/zones/foreman.de.rev
  434  nano /etc/fstab
  435  nano /etc/exports
  ```

## Configure Foreman for external DNS management

