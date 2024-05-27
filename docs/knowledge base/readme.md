**| [Knowledge Base](https://ji-podhead.github.io/RHEL_9_Foreman_Guide/knowledge%20base)|[Install](https://ji-podhead.github.io/RHEL_9_Foreman_Guide/installation%20(katello%2Cdiscovery%2Cdhcp%2Ctftp)) | [Discovery and Provisioning](https://ji-podhead.github.io/RHEL_9_Foreman_Guide/discovery%20and%20provisioning) | [libvirt](https://ji-podhead.github.io/RHEL_9_Foreman_Guide/libvirt) | [proxmox](https://ji-podhead.github.io/RHEL_9_Foreman_Guide/proxmox) |** 

---
- this might be a little redundant at the moment, but i thought its better to explain everything step after step, so the diagramms wont get to large
### DHCP
![dhcp](https://github.com/ji-podhead/RHEL_9_Foreman_Guide/blob/main/img/dhcp.png?raw=true)

### Subnets and VLAN
![subnets&vlan](https://github.com/ji-podhead/RHEL_9_Foreman_Guide/blob/main/img/subnets%20und%20vlan.png?raw=true)
> - MAAS:
> 	- Focus: Direct management of VLANs and subnets within the platform itself.
> 	- Functionality: Allows creation and management of multiple VLANs, supporting both tagged and untagged VLANs on managed switches. Each fabric has a default VLAN, with additional VLANs added for logical separation within the same physical infrastructure.
> - Foreman:
> 	- Focus: Primarily focused on provisioning hosts and configuring their network settings.
> 	- Functionality: Can provision hosts across various subnets but does not directly manage VLANs. Requires separate network-level configuration for creating and managing VLANs.
> In essence, MAAS offers integrated VLAN and subnet management within its environment, whereas Foreman focuses on host provisioning and network configuration, leaving VLAN management to external network administration.

### DNS
![dns](https://github.com/ji-podhead/RHEL_9_Foreman_Guide/blob/main/img/dns.png?raw=true)

- [do routers have dns?](https://superuser.com/questions/1715361/do-routers-have-a-dns-server)
... 
> - most SOHO routers have a built-in DNS server to act as a cache. It's not a mandatory "router" feature though – enterprise networks would run their DNS on a separate system instead.
> - **If this is so, then I guess that DNS server would just be another cache similar to the one in Windows...or is it a more advanced DNS server?**
> 	- It varies between products. Talking about SOHO routers, the router's own DNS server is pretty much always just a caching proxy and actual name resolution relies on forwarding requests to an upstream resolver; no root hints involved.
> 	- But in addition to that, it is also quite common for the router to be authoritative for some "local" domain (like .lan or .home or .dlink) which contains hostnames for your LAN hosts. This integrates with the router's DHCP service, collecting hostnames that devices provide in their lease requests. It may even support static entries, though in SOHO routers it's rarely anything more than a single 'A' record per name.
- why does foreman require dns?
> - To provision with Foreman, the DNS domain of the router is essential because Foreman requires defined domain names for every host in the network. These domain names are crucial for managing A-, AAAA-, and PTR resource records. Even if Foreman doesn't manage your DNS servers, you still need to create at least one domain and assign it. Domains are part of the naming conventions that Foreman uses for hosts, for example, a host named test123 in the example.com domain has the fully qualified domain name test123.example.com.
> - During the DNS record creation process, Foreman performs conflict DNS answers to ensure that the hostname is not actively being used. This check is performed against one of the following DNS servers:
>  	- The system-wide resolver, if under Administer > Settings > Query local nameservers the option true is enabled.
>  	- The nameservers defined in the subnet agreement with the host.
>  	- The authoritative NS-records derived from the SOA (Start of Authority) of the domain name associated with the host.
> - Therefore, knowing the domain name of the router is crucial for successful provisioning with Foreman to ensure that all DNS queries are handled correctly and no conflicts arise. This is particularly important when Foreman attempts to dynamically assign IP addresses or checks existing DNS entries to ensure that no IP addresses are duplicated.

### hosts file
> This diagram outlines the differences in the process of resolving a domain name using the hosts file versus a DNS server:
> - This sequence diagram simplifies the process for illustrative purposes. The actual process may involve additional steps, such as DNS caching and recursive queries, depending on the network infrastructure and protocols in use.
![host_file](https://github.com/ji-podhead/RHEL_9_Foreman_Guide/blob/main/img/hosts_file.png?raw=true)

> - Using the Hosts File:
>	- The client's web browser first checks the hosts file for an entry corresponding to the domain name (www.example.com).
> 	- If an entry is found, the browser uses the IP address listed in the hosts file to connect to the website.
> 	- This method bypasses DNS resolution entirely, relying solely on local entries.
> - Using a DNS Server:
> 	- If the hosts file does not contain an entry for the domain, the browser queries a DNS server for the IP address associated with www.example.com.
> 	- The DNS server returns the IP address, which the browser then uses to establish a connection to the website.
> 	- This method relies on external DNS services to resolve domain names.
> The key difference lies in the source of the IP address used for accessing the website:
> - Hosts File: Provides static mappings of domain names to IP addresses on a local machine, taking precedence over DNS when resolving domain names locally.
> - DNS Server: Acts as a global directory service, dynamically resolving domain names to IP addresses based on the current state of the internet.

### PXE and TFTP
> This diagram outlines the basic steps involved in the PXE and TFTP boot process:
![pxe](https://github.com/ji-podhead/RHEL_9_Foreman_Guide/blob/main/img/PXE_and_TFTP.png?raw=true)
> - ***Client Starts Booting:*** The client device initiates the boot process.
> - ***Sends DHCPDISCOVER:*** The client sends a DHCPDISCOVER packet to discover available DHCP servers.
> - ***Receives DHCPOFFER:*** The DHCP server responds with a DHCPOFFER packet, providing the client with an IP address and the address of the TFTP server.
> - ***Requests Boot Image:*** The client requests the boot image from the TFTP server.
> - ***Sends Boot Image:*** The TFTP server sends the boot image to the client.
> - ***Executes Boot Image:*** The client executes the boot image, initiating the boot process from the network.

## Foreman Smartproxy  & Network Configuration Process
> This diagram provides a visual representation of the network configuration process, detailing how a client PC interacts with various components such as VLAN, DNS Server, DHCP in Router, and Storage during the boot process
![smartproxy](https://github.com/ji-podhead/RHEL_9_Foreman_Guide/blob/main/img/smartproxy.png?raw=true)

# Network Bridge
**A [network bridge](https://de.wikipedia.org/wiki/Bridge_(Netzwerk)) is a computer networking device that creates a single, aggregate network from multiple communication networks or network segments. This function is called network bridging** 

# Lifecycle Management

>Lifecycle management refers to the process of overseeing the entire lifespan of a product, from its inception through various stages of development, deployment, operation, and ultimately to its retirement or disposal. This comprehensive approach ensures that every aspect of the product's journey is managed effectively, optimizing the development process, enhancing product quality, reducing time-to-market, and increasing profitability.
>
> In the context of IT systems, lifecycle management encompasses the administration of a system from provisioning, through operations, to retirement. It allows for the reliable creation of systems in an automated and scalable manner, tracking and accounting for all systems, assets, and subscriptions, ensuring consistency across the system's lifecycle, and decommissioning systems and resources when they are no longer needed.
>
> Application lifecycle management (ALM) specifically addresses the management of applications throughout their lifecycle, from requirements gathering through development, testing, deployment, and maintenance. ALM is crucial for ensuring quality and traceability in the application development process, facilitating efficient and effective management of application lifecycles.
>
> Overall, lifecycle management is essential for improving product quality, accelerating time-to-market, ensuring compliance in regulated industries, and providing visibility into the development process. Whether dealing with physical products, software applications, or IT systems, lifecycle management plays a vital role in guiding business decisions and strategies from conception to retirement.
```Bash
+--------+   +--------------+   +---------------+   +------------+   +-----------+   +-------------+   +---------------------+   +------------+   +-----------+     
| Start  |-->| Provisioning |-->| Configuration |-->| Deployment |-->| Operation |-->| Maintenance |-->| Maintenance Updates |-->| Retirement |-->|   End     |
+--------+   +--------------+   +---------------+   +------------+   +-----------+   +-------------+   +---------------------+   +------------+   +-----------+
```


    

> This flowchart starts with the initiation of the lifecycle management process and progresses through the following stages:
>    - ***Provisioning:*** Setting up the necessary resources and environments for the system or application.
>    - ***Configuration:*** Configuring the system or application according to specifications.
>    - ***Deployment:*** Deploying the system or application into the production environment.
>    - ***Operation:*** Running the system or application in the operational environment.
>    - ***Maintenance:*** Performing regular checks and updates to ensure the system or application remains functional and secure.
>    - ***Maintenance Updates:*** Applying updates and patches during the maintenance phase.
>    - ***Retirement:*** Decommissioning the system or application once it is no longer needed.
>    - ***End:*** Completion of the lifecycle management process. -> `END OF LIFE`

---
> Both Puppet and Katello are used for lifecycle management, but their roles and areas of application differ.

***Puppet*** is traditionally used for host configuration management. It allows administrators to define and manage the configuration of computers using a simple declarative language syntax. With Puppet, you can automate and manage software configuration, filesystems, packages, services, and much more on your hosts. However, support for Puppet in the context of Red Hat Satellite (now part of Red Hat Smart Management) has been discontinued with the introduction of Satellite 7.0. Users who previously used Puppet within Satellite have two options: either migrate to Ansible or integrate Puppet outside of the Satellite environment .

***Katello***, part of Red Hat Satellite, offers features for content lifecycle management, including package sources, repository management, and lifecycle management. With Katello, you can manage the content needed for your infrastructure and distribute this content to your hosts. The use of the Katello Agent enabled communication between clients and the Satellite server to initiate tasks such as performing package updates or installations on the clients. However, with the introduction of remote execution functionality in Satellite 6.2, there was the possibility to use SSH for communication, reducing the need for the Katello Agent. Transitioning to a "goferless" configuration (without the Katello Agent) is recommended in future versions of Satellite .

---




---
**| [Knowledge Base](https://ji-podhead.github.io/RHEL_9_Foreman_Guide/knowledge%20base)|[Install](https://ji-podhead.github.io/RHEL_9_Foreman_Guide/installation%20(katello%2Cdiscovery%2Cdhcp%2Ctftp)) | [Discovery and Provisioning](https://ji-podhead.github.io/RHEL_9_Foreman_Guide/discovery%20and%20provisioning) | [libvirt](https://ji-podhead.github.io/RHEL_9_Foreman_Guide/libvirt) | [proxmox](https://ji-podhead.github.io/RHEL_9_Foreman_Guide/proxmox) |** 
