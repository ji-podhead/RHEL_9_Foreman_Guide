
## *Foreman in a nested VM* managing external DNS & DHCP with Dynamic Updates 
> - we will install & configure a Foreman-machine running inside a Proxmox-libvirt VM
> - we will install & configure our DHCP & DNS on Debian in a seperate libvirt-VM
> - we will configure our DHCP get managed by Foreman and share its leases
> - we will configure Foreman to manage our external DHCP and DNS
> - how to debug your servers and monitor the network 
>  - Discovery process walktrough   

---

### Preview


   <img src="https://github.com/ji-podhead/RHEL_9_Foreman_Guide/blob/main/docs/nestedVM_with_external_DHCP&DNS/images/final_discovery_nestedv_flowchart.png?raw=true" align="center" width="500" />

### Specs used in this Guide

| ***Description*** | ***Type***| 
|--------------------|-----------|
| Host | 192.168.122.1
| Foreman-Proxy | 192.168.122.20 |
| Foreman-FQDN | foreman.de |
| DNS & THCP-Server | 192.168.122.7 |
| Subnet |  `adress`  192.168.122.0  <br> `range` 192.168.122.1 192.168.122.254 <br> `option routers` 192.168.122.7 <br> `option broadcast-address` 192.168.122.255 |


---
### Setup a test machine for pxe boot

- make sure to add your NIC to Boot-Options

![pxe-test-machine](https://github.com/ji-podhead/RHEL_9_Foreman_Guide/blob/main/docs/nestedVM_with_external_DHCP&DNS/images/foreman_nestedVM_pxeboot.png?raw=true)


###  DHCP & DNS installation & configuration steps
- create a seperate machine 
  - I was to lazy and directly installed on my Proxmox-Machine, which is stupid:
	- DNS holds a huge risk when misconfigured or attacked
	- if your DNS starves, it will also starve all your Proxmox-stuff and might even damage the Filesystem 
- setup your Debian-based `Bind9 DNS` and `ISC-DHCP`
	- I coulnd get my DHCP on my Foreman Machine to work with the provided Proxmox-NIC
- **Foreman wont register your machines, even if they have a valid tftp connection, unless you share the leases of DHCP!** 
- Therefor these procedures have to get accomplished:
	- 1.  [Configuring an external DHCP server to use with Foreman server](https://docs.theforeman.org/nightly/Installing_Server/index-foreman-deb.html#configuring-an-external-dhcp-server_foreman)
    
	- 2.  [Configuring Foreman server with an external DHCP server](https://docs.theforeman.org/nightly/Installing_Server/index-foreman-deb.html#Configuring_Server_with_an_External_DHCP_Server_foreman)
- both procedures will be covered in this  guide


---

 ***Please proceed with the DNS section of my [DNS-Network Guide](https://ji-podhead.github.io/Network-Guides/DNS/install/):***
 - All DNS-related topics needed are explained in detail here
> - [Knowledge Base ](https://ji-podhead.github.io/Network-Guides/DNS/Knowledge%20Base)
> - [Install & Config](https://ji-podhead.github.io/Network-Guides/DNS/install)
> - [Test & Debug](https://ji-podhead.github.io/Network-Guides/DNS/testAndDebug)
> - [Attack- Vectors and Scenario](https://ji-podhead.github.io/Network-Guides/DNS/attackVectorsAndScenario)
> - [Security & Protection](https://ji-podhead.github.io/Network-Guides/DNS/protection)


---


## Dynamic Updates & Shared Leases
> `/etc/bind/named.conf`
***named.conf***
```yaml
include "/etc/bind/named.conf.options";
include "/etc/bind/named.conf.local";
include "/etc/bind/named.conf.default-zones";
```
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

---
```
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
## Initialize Foreman with Discovery 
- set managed DNS & DHCP to false
```Bash
foreman-installer --foreman-proxy-dns false --foreman-proxy-dns-managed false --foreman-proxy-dhcp true --foreman-proxy-dhcp-managed true --foreman-proxy-dhcp-range "192.168.122.1 192.168.122.100" --foreman-proxy-dhcp-gateway 192.168.122.7 --foreman-proxy-dhcp-nameservers 192.168.122.7 --foreman-proxy-tftp true --foreman-proxy-tftp-managed true --foreman-proxy-tftp-servername 192.168.122.20
```

## *Foreman in a nested VM* managing external DNS & DHCP with Dynamic Updates 
> - we will install & configure a Foreman-machine running inside a Proxmox-libvirt VM
> - we will install & configure our DHCP & DNS on Debian in a seperate libvirt-VM
> - we will configure our DHCP get managed by Foreman and share its leases
> - we will configure Foreman to manage our external DHCP and DNS
> - how to debug your servers and monitor the network 
>  - Discovery process walktrough   

---

### Preview


   <img src="https://github.com/ji-podhead/RHEL_9_Foreman_Guide/blob/main/docs/nestedVM_with_external_DHCP&DNS/images/final_discovery_nestedv_flowchart.png?raw=true" align="center" width="500" />

### Specs used in this Guide

| ***Description*** | ***Type***| 
|--------------------|-----------|
| Host | 192.168.122.1 |
| PXE-Test-Machine | 192.169.122.138 |
| Foreman-Proxy | 192.168.122.20 |
| Foreman-FQDN | foreman.de |
| DNS & THCP-Server | 192.168.122.7 |
| Subnet |  `adress`  192.168.122.0  <br> `range` 192.168.122.1 192.168.122.254 <br> `option routers` 192.168.122.7 <br> `option broadcast-address` 192.168.122.255 |


---
### Setup a test machine for pxe boot

- make sure to add your NIC to Boot-Options

![pxe-test-machine](https://github.com/ji-podhead/RHEL_9_Foreman_Guide/blob/main/docs/nestedVM_with_external_DHCP&DNS/images/foreman_nestedVM_pxeboot.png?raw=true)
---

###  DHCP & DNS installation & configuration steps
- create a seperate machine 
  - I was to lazy and directly installed on my Proxmox-Machine, which is stupid:
	- DNS holds a huge risk when misconfigured or attacked
	- if your DNS starves, it will also starve all your Proxmox-stuff and might even damage the Filesystem 
- setup your Debian-based `Bind9 DNS` and `ISC-DHCP`
	- I coulnd get my DHCP on my Foreman Machine to work with the provided Proxmox-NIC
- **Foreman wont register your machines, even if they have a valid tftp connection, unless you share the leases of DHCP!** 
- Therefor these procedures have to get accomplished:
	- 1.  [Configuring an external DHCP server to use with Foreman server](https://docs.theforeman.org/nightly/Installing_Server/index-foreman-deb.html#configuring-an-external-dhcp-server_foreman)
    
	- 2.  [Configuring Foreman server with an external DHCP server](https://docs.theforeman.org/nightly/Installing_Server/index-foreman-deb.html#Configuring_Server_with_an_External_DHCP_Server_foreman)
- both procedures will be covered in this  guide


---

 ***Please proceed with the DNS section of my [DNS-Network Guide](https://ji-podhead.github.io/Network-Guides/DNS/install/):***
 - All DNS-related topics needed are explained in detail here
> - [Knowledge Base ](https://ji-podhead.github.io/Network-Guides/DNS/Knowledge%20Base)
> - [Install & Config](https://ji-podhead.github.io/Network-Guides/DNS/install)
> - [Test & Debug](https://ji-podhead.github.io/Network-Guides/DNS/testAndDebug)
> - [Attack- Vectors and Scenario](https://ji-podhead.github.io/Network-Guides/DNS/attackVectorsAndScenario)
> - [Security & Protection](https://ji-podhead.github.io/Network-Guides/DNS/protection)


---


## Dynamic Updates & Shared Leases
> `/etc/bind/named.conf`
***named.conf***
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

---

```
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

## Initialize Foreman with Discovery 
- set managed DNS & DHCP to false
```Bash
foreman-installer --foreman-proxy-dns false --foreman-proxy-dns-managed false --foreman-proxy-dhcp true --foreman-proxy-dhcp-managed true --foreman-proxy-dhcp-range "192.168.122.1 192.168.122.100" --foreman-proxy-dhcp-gateway 192.168.122.7 --foreman-proxy-dhcp-nameservers 192.168.122.7 --foreman-proxy-tftp true --foreman-proxy-tftp-managed true --foreman-proxy-tftp-servername 192.168.122.20
```
