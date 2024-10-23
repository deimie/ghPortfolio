---
layout: single
title:  "Building Blue Team Homelab Part 3 - DFIR"
date:   2024-09-15
categories: Homelab
---

For part 3, we will be setting up the security team portion of our network. We will install VMs to handle malware analysis, DFIR, as well as a SIEM. 

## Parrot OS
For now we will start with a machine to handle DFIR (Data Forensics and Incident Response). In this case, I will use a flavor of Linux called [Parrot OS](https://parrotsec.org/), but there are other great OS's preloaded with DFIR tools similar to Parrot, and this specific OS is not an exact requirement. In my case, I will also be using the Security distribution of Parrot.

The installation should be straightforward and default settings with minimum specs will be fine. Once Parrot is fully installed, we will need to make some configuration changes to inccorporate this computer into our network.

### Basic Configuration
Once the install is completed, we will change the network interface to VLAN 15, which is our security VLAN. We will also assign a static IP address and our domain name to this machine. After a system restart, the changes should stick.

At the top, navigate to ```System > Preferences > Internet and Network > Advanced Network Configuration```. We will set the static IP address, the gateway, and assign our domain name to this address.

![parrotNetConfig.jpg failed to load.]({{ site.baseurl }}/assets/images/homelab/parrotNetConfig.jpg)

Next, we will assign this machine's IP address to our domain name using the DNS resolver in the pfSense GUI. Navigate to ```Services > DNS Resolver > General Settings > Host Override``` and configure the domain with the hostname and the machine's IP address. 

![dnsHostOverride.jpg failed to load.]({{ site.baseurl }}/assets/images/homelab/dnsHostOverride.jpg)

We can use the ping tool in ```Diagnostics > Ping``` to check if this change is working.

![pingDomain.jpg failed to load.]({{ site.baseurl }}/assets/images/homelab/pingDomain.jpg)

Now that this is working, we are done connecting this machine to the network and it is ready to use for DFIR purposes. Next up, we will work on the isolated network and setting up the computer that will handle MA on that VLAN.