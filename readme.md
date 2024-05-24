
# Foreman 3.10 + Puppet + Katello + Discovery Plugin-Installation +  PXE + local-DHCP&TFTP Guide for RHEL 9
 
 In this Guide i will show you how to install Forman with puppet, katello and discovery plugin.
 You will also learn how to install and setup DHCP- and TFTP-Server.
 I will also show you how to setup Foreman and how to use the Foreman Boot Image via PXE
You will be ready to discover and provision your physical servers and workstations after following this Guide.
 > ***before we start:***
 >  - foreman comes without its own dhcp/tftp unlike MAAS, Tinkerbell, etc
>    - you either need to have external dhcp, or you need to install the servers locally
 >  - we will install and we will use Foreman on a single node ***without external DHCP***
> - Its demanded that you install Foreman with Katello on a ***freshly provisioned machine***
>  -  we wont use  ***Smartproxy DNS*** since its not required if using a local DHCP 
>-  we install Discovery Plugin before setting up TFTP because we have less work
> -  make sure that you have a ***Backup*** before using the Installer
>    - *especially if you have set up Foreman successfully before*
>    - you can make backup by using:
>       - img (dd, gparted)
>       - rsync (standalone, or better: ***rsnapshot***) 
## Preperation

- make sure you have a static hostname (mine is `my_hostname`)

***switch to root  because its easier:***
``` Bash
$ su root
```

***get your  NIC's IP and Name:***
```Bash
# ifconfig
```

>```
> enp2s0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
>       inet 192.168.2.100  netmask 255.255.255.0  broadcast 192.168.2.255
>
>```.

> - my NIC is enp2s0 and my IP is 129.168.2.100:


***find your NIC's DNS-Server's IP and Domain***
- we  need this for the hosts mapping
- The Domain of your Router should be printed on it, but we can also find it out via console:
 ```Bash
 # nmcli device show enp2s0 | grep IP4.DNS
 ```
> ```
> # 									DNS-Server-IP:
> IP4.DNS[1]:                             192.168.2.1
> ```
```Bash
# nslookup 192.168.2.1
 ```
>```
> 1.2.168.192.in-addr.arpa	name = speedport.ip.
> ```
***edit the hosts file***
 
- edit `/etc/hosts`
	-  the Domain for the host mapping should  be:
		-  <host name+routers domain>
```
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

192.168.2.100 my_hostname.speedport.ip
```
***firewall settings:***
```Bash
# firewall-cmd --add-port="5646/tcp"
```
```Bash
# firewall-cmd \
--add-port="5647/tcp" \
--add-port="8000/tcp" \
--add-port="9090/tcp"
```
```Bash
# firewall-cmd \
--add-service=dns \
--add-service=dhcp \
--add-service=tftp \
--add-service=http \
--add-service=https \
--add-service=puppetmaster
```
```Bash
# firewall-cmd --runtime-to-permanent
```
> ***>> check if it works <<***
 >  ```Bash
 >  # firewall-cmd --list-all
 >  ```
 > should ouput:
 > ```markdown
>public (active)
>  target: default
>  icmp-block-inversion: no
>  interfaces: enp2s0
>  sources: 
>  services: cockpit dhcp dhcpv6-client dns http https mdns puppetmaster ssh tftp
>  ports: 5646/tcp 5647/tcp 8000/tcp 9090/tcp
>  protocols: 
>  forward: yes
>  masquerade: no
>  forward-ports: 
> source-ports: 
> icmp-blocks: 
>  rich rules: 
> ```
## Install

***get the repos***
```Bash
# dnf install https://yum.theforeman.org/releases/3.10/el9/x86_64/foreman-release.rpm
# dnf install https://yum.theforeman.org/katello/4.12/katello/el9/x86_64/katello-repos-latest.rpm
# dnf install https://yum.puppet.com/puppet7-release-el-9.noarch.rpm
```





***install  foreman 3.10 with katello plugin***
```Bash
# dnf update
# dnf install foreman-installer-katello
# foreman-installer --scenario katello
```
```
...
  Success!
  * Foreman is running at https://my_hostname.speedport.ip
      Initial credentials are admin / LjSnHiw9f96PTTen
  * To install an additional Foreman proxy on separate machine continue by running:

      foreman-proxy-certs-generate --foreman-proxy-fqdn "$FOREMAN_PROXY" --certs-tar "/root/$FOREMAN_PROXY-certs.tar.gz"
  * Foreman Proxy is running at https://my_hostname.speedport.ip:9090

The full log is at /var/log/foreman-installer/katello.log
```
---
> **we connect to foreman dashboard by using**
> URL: https://my_hostname.speedport.ip
> user: admin
> pass:  LjSnHiw9f96PTTen`


---
***install the Discovery Plugin***
```Bash
# foreman-installer --enable-foreman-plugin-discovery
```
```
...
  Success!
  * Foreman is running at https://my_hostname.speedport.ip
  * To install an additional Foreman proxy on separate machine continue by running:

      foreman-proxy-certs-generate --foreman-proxy-fqdn "$FOREMAN_PROXY" --certs-tar "/root/$FOREMAN_PROXY-certs.tar.gz"
  * Foreman Proxy is running at https://my_hostname.speedport.ip:9090

The full log is at /var/log/foreman-installer/katello.log
```
---
> ***>> check if it worked <<***
> ```Bash
> # dnf repolist enabled
>```
> ```
> ...
> foreman                   Foreman 3.10
> foreman-plugins           Foreman plugins 3.10
> katello                   Katello 4.12
> pulpcore                  pulpcore: Fetch, Upload, Organize, and Distribute Software Packages.
> puppet7                   Puppet 7 Repository el 9 - x86_64
>```


> ****(optional)* delete old/wrong repo:***
> - edit the foreman.repo file and remove the flawed ones:
> ```Bash
> # dnf clean all
> # dnf install nano
> # sudo nano /etc/yum.repos.d/foreman.repo
> # sudo dnf clean all
> # sudo dnf makecache
> ```
> 
## DHCP 
***Install:***
```Bash
# dnf install dhcp-server -y
```

***Config:***
- we add a Subnet
  - we choose a Range of 100 (huge Networks can be unnecessary security Risk)
  
```Bash 
# sudo nano /etc/dhcp/dhcpd.conf
```


> ```Bash 
> ...
> # speedport.ip
>subnet 192.168.2.0 netmask 255.255.255.0 {
 > pool
>  {
>    range 192.168.2.101 192.168.2.200;
>  }
>  option subnet-mask 255.255.255.0;
>  option routers 192.168.2.100;
>}
> ```
- Now we we can enable the dhcp service
	-  if this this fails you most likely have wrong subnet or firewall settings
 > ```Bash
 > # sudo systemctl enable --now dhcpd
>```

 > ****(optional)* check if dhcp server is already installed and running***
>```Bash
> # nmap -sU 127.0.0.1 -p 67
> ```
> 
>  ```markdown
>  # if  its not installed or not running:
> ...
> PORT   STATE  SERVICE
> 67/udp closed dhcps
> ...
> ```
>  ```markdown
> #  if up and running:
> ...
> PORT   STATE         SERVICE
> 67/udp open|filtered dhcps
> ...
> ```
> of course you can check systemctl as well, but  since we dont know the name of the service we   just check the port directly (DHCP is Port 67 followed by TFTP port 68)
> you can also use telnet, lsof, etc

## TFTP

```Bash
# sudo dnf install tftp-server -y
```
***check if Discovery-Plugin created the Boot-image Files:***
- there should be a `/var/lib/tftpboot/boot/fdi-image` dir that holds the  `vmlinuz` and `initrd` files
- you also need to create a config file:   `nano /var/lib/tftpboot/pxelinux.cfg/default`
   -  the **user has to be nobody** (system-user) and it should be **fully writable**
   
 ```Bash
 #  nano /var/lib/tftpboot/pxelinux.cfg/default
   ```
   
>```
> default menu.c32
> timeout 300
> label ForemanBootImage
>  menu label ^Foreman Boot Image
> kernel /path/to/your/boot/image/vmlinuz
> append initrd=/path/to/your/boot/image/initrd.img root=/dev/nfs nfsroot=:192.168.0.1:/var> > /lib/tftboot/boot ip=dhcp
> ```

```Bash
# sudo chmod -R 777 /var/lib/tftpboot
# sudo chown -R nobody: /var/lib/tftpboot
```
   - Change tftpboot dir if required:
```bash
$ nano /usr/lib/systemd/system/tftp.service
```

>```
> [Unit]
> Description=Tftp Server
> Requires=tftp.socket
> Documentation=man:in.tftpd
> [Service]
> ExecStart=/usr/sbin/in.tftpd -s /var/lib/tftpboot
> StandardInput=socket
> [Install]
> Also=tftp.socket
>```

- not sure if this was required:
  
>```Bash
> sudo nano /etc/xinetd.d/tftp
>```

>```
> service tftp
> {
> socket_type             = dgram
> protocol                = udp
> wait                    = yes
> user                    = root
> server                  = /usr/sbin/in.tftpd
> server_args             = -s /var/lib/tftpboot
> disable                 = no 						# needs to be "no"
> per_source              = 11
> cps                     = 100 2
> flags                   = IPv4
> }
>```



-  tftp service can be activated by using systemctl enable tftp (not xintetd)!
>```Bash
> # systemctl enable tftp
>```

## Update Foreman
- we set managed dns to false: `--foreman-proxy-dns-managed false \`
```Bash
#  foreman-installer \
--foreman-proxy-dns true \
--foreman-proxy-dns-managed false \
--foreman-proxy-dhcp true \
--foreman-proxy-dhcp-managed true \
--foreman-proxy-dhcp-range "192.168.2.101 192.168.2.200" \
--foreman-proxy-dhcp-gateway 192.168.2.100 \
--foreman-proxy-dhcp-nameservers 192.168.2.100 \
--foreman-proxy-tftp true \
--foreman-proxy-tftp-managed true \
--foreman-proxy-tftp-servername 192.168.2.100
```



