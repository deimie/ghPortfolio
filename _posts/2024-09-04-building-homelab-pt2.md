---
layout: single
title:  "Blue Team Homelab Part 2 - Configuring the Firewall"
date:   2024-09-04
categories: Homelab
---

In this part of building the homelab, I will be installing and configuring a pfSense firewall.

### Creating pfSense VM
For the firewall, I am using a free open-source software called [pfSense](https://www.pfsense.org/). It can be downloaded as a disc image (ISO) to be run in vmware. For installation, all default settings will suffice.

In order to configure the pfSense firewall, I had to edit the LAN interface’s IP address to match the subnet address that I had specified in the vmware Virtual Network Editor. To do this, select option 2. Next, choose no for DHCP assigning the address for us, as we will set the IP address ourselves. I will use 10.0.1.1, which will be the gateway to access the pfSense configuration GUI from another machine that is connected to the LAN. For this project, IPv6 will not be utilized, so select no for all options asking about it. Lastly, you will be prompted if you want to set up a DHCP server to assign addresses to machines that connect to this interface. I allowed the DHCP server to assign addresses between 10.0.1.30 to 10.0.1.100.


![pfSenseOS.jpg]({{ site.baseurl }}/assets/images/homelab/pfSenseOS.jpg)

### Accessing the GUI
Next, I set up a Ubuntu virtual machine to test the firewall and manage the firewall configuration using the GUI. Any operating system with a web browser will work for this step. This box needs its NIC to be the same as the pfSense LAN’s NIC, which is VMnet1 in this case. Now the GUI interface can be accessed by inputting the gateway address into a web browser. The default credentials are “admin” and “pfsense”.

![pfSensePortal.jpg]({{ site.baseurl }}/assets/images/homelab/pfSensePortal.jpg)

### Interface Configuration
At the top navbar go to Interfaces/Interface Assignments. Here we will configure all of the interfaces within the firewall by assigning each interface a descriptive name and their corresponding static IPv4 gateway addresses, referencing the network topology table we made earlier.

![fwInterfaces.jpg]({{ site.baseurl }}/assets/images/homelab/fwInterfaces.jpg)

**Basic Configuration**\
Go to System/General Setup. Here, we will set a few basic configurations:
- Hostname: Set to whatever you’d like
- Domain: I will keep the default “home.arpa” but this can be any domain you’d like
- DNS Servers: I will use Google’s DNS service with 8.8.8.8 and 8.8.4.4, but feel free to use any DNS server you would like
- Timezone: Etc/UTC

Go to System/Advanced/Admin Access.
- Protocol: HTTPS (SSL/TLS)
- Secure Shell Server: Enable

**Firewall Rules**\
Go to Firewall/Rules.

> WAN - Default configuration will block private networks and bogon networks (bogon = fake IP addresses). Leave these as is.

![wanRules.jpg]({{ site.baseurl }}/assets/images/homelab/wanRules.jpg)

> Management - Default configuration is mostly fine, but I will disable the IPv6 rule here.

### IMAGE NEEDED HERE

> Corporate WAN - Here, I will allow all traffic toward the Corporate LAN and the Corporate WAN from all sources.

### IMAGE NEEDED HERE

> Corporate LAN - This interface will have the exact same rules as the corporate WAN interface. This is so we can simulate access to the fake internet. In a real environment, these would be different.

### IMAGE NEEDED HERE

> Security - The security network should not have access to WAN, Management, or Corporate WAN. All other connections are fine.

### IMAGE NEEDED HERE

> Isolation LAN - The isolation network shouldn't have any access in or outbound. We will deny all traffic in and out.

### IMAGE NEEDED HERE