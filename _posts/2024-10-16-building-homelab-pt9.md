---
layout: single
title:  "Building Blue Team Homelab Part 9 - SIEM Installation"
date:   2024-10-20
categories: Homelab
---

We will go through the process of installing a SIEM and configuring it to listen to network traffic.

## Security Onion
### Installation
For our SIEM, we will be using an open-source software called [Security Onion](https://github.com/Security-Onion-Solutions/securityonion/blob/2.4/main/DOWNLOAD_AND_VERIFY_ISO.md). Since we will ultimately be using the Standalone version of Security Onion, the bare minimum specs are: 200 GB of storage, 4 CPU cores, and 16 GB of RAM.

Additionally, we will need to create one more VLAN in vmware since this SIEM will need access to 2 NICs. The first NIC will be our security VLAN 15, and the second one I will designate to a new VLAN 16. The purpose second NIC will be explained after intallation.

![newVLAN16.jpg failed to load.]({{ site.baseurl }}/assets/images/homelab/buildPart9/newVLAN16.jpg)

Now add the interface for your new VLAN to both the pfSense VM and the security onion VM. This is roughly how your box settings should look, but I'm using 18GB of RAM to slightly improve performance since 16GB is the bare minimum.

![onionSpecs.jpg failed to load.]({{ site.baseurl }}/assets/images/homelab/buildPart9/onionSpecs.jpg)

> Make sure the pfSense firewall is on throughout the installation, since internet connection is required and the security VLAN that this SIEM will sit on has internet access.

Continue through the installation of the OS with standard settings until prompted to choose the installation type, in which you will choose *Standalone*.

![onionStandalone.jpg failed to load.]({{ site.baseurl }}/assets/images/homelab/buildPart9/onionStandalone.jpg)

You can continue with default settings until prompted to select the NIC for management. You want to select the NIC that is associated to your security VLAN. To do this open the VM settings of security onion.

![onionSelectNIC.jpg failed to load.]({{ site.baseurl }}/assets/images/homelab/buildPart9/onionSelectNIC.jpg)

Find your security VLAN and under *Advanced* check the MAC address.

![onion-NIC-MAC.jpg failed to load.]({{ site.baseurl }}/assets/images/homelab/buildPart9/onion-NIC-MAC.jpg)

Next, select *Static* and configure a valid static IP address, the default gateway, and DNS nameservers in accordance to your security VLAN. In my case: 10.0.15.17/24, 10.0.15.254, and 10.0.15.254 respectively. Then set the DNS search domain to the same domain as we've used before (homelab.lan in my case).

Continue with default settings, then select the monitor interface (there should only be one option).

![onionMonitorInterface.jpg failed to load.]({{ site.baseurl }}/assets/images/homelab/buildPart9/onionMonitorInterface.jpg)

Now enter an email address of your choice for the SOC console user, but using your own domain.

![onionAdminAccount.jpg failed to load.]({{ site.baseurl }}/assets/images/homelab/buildPart9/onionAdminAccount.jpg)

Now to access the web interface, select *Other* and choose your own FQDN for the SIEM. I will use *hlsiem.homelab.lan*.

![onionWebInterface.jpg failed to load.]({{ site.baseurl }}/assets/images/homelab/buildPart9/onionWebInterface.jpg)

After continuing through, you will reach a page asking to enter an IP address/range to allow access. You can either set this to your home network subnet if you want to use your host machine browser to access, or one of the VLAN subnets. I will use my home network address for simplicity, but in a real production environment, it would of course be one of the corporate networks that accesses the web interface.

![webInterfaceIP.jpg failed to load.]({{ site.baseurl }}/assets/images/homelab/buildPart9/webInterfaceIP.jpg)

Finally, continue through until installation begins. This installation will take a little while.

### Web Interface

Before you can connect to the web interface, it is important that the FQDN you set in configuration actually resolves to the IP of the SIEM. If you are connecting on your host machine, add an entry in your hosts file (just like we did in part 7) that associates your FQDN to the IP of the security onion box. 

If you would like to connect to the web interface using a VM, add a DNS entry in your pfSense web configurator under ```Services > DNS Resolver > Host Overrides```.

Now let's connect to the web interface on whichever IP address you allowed. If the web interface isn't immediately working, run ```sudo so-status``` in the security onion console and ensure all the services are running.

![onionSOCInterface.jpg failed to load.]({{ site.baseurl }}/assets/images/homelab/buildPart9/onionSOCInterface.jpg)

### pfSense Port Mirroring
Overall, this SIEM will monitor the network in two ways. Firstly, we will have agents deployed that collect logs on individual end devices. Additionally, the SIEM will also collect overall network traffic, which is what we will do right now. To do this, we will utilize a method on the firewall called port mirroring. This will allow us to copy all the network traffic on one network interface, and copy it to another interface without any interference to the network.

This means that we will have one additional interface in pfSense which all other network interfaces will copy their traffic to. This is why we created a VLAN at the start of this part, which was VLAN 16 for me.

Now, in the pfSense web configurator, navigate to ```Interfaces > Assignments``` and assign the new interface. Enable it, give it a name, and save without doing anything else.

![spanPort.jpg failed to load.]({{ site.baseurl }}/assets/images/homelab/buildPart9/spanPort.jpg)

Now in ```Interfaces > Assignments > Bridges```, hit add. Select your Corporate LAN under *Member Interfaces*, and the Span port under *Span Port*. 

![bridgeInterfaces.jpg failed to load.]({{ site.baseurl }}/assets/images/homelab/buildPart9/bridgeInterfaces.jpg)

Save this, and now we can begin testing.

### Testing the SIEM
Now, boot up your kali linux box and the ubuntu web server. I'm going to run a few different commands from kali towards the web server, but feel free to mess around and try anything.

* ```ping 10.0.11.10``` - Simple ping - Low severity alert
* ```sqlmap -u "http://10.0.11.10/"``` - Sqlmap scan that checks for sql vulnerabilities - Medium severity alert
* ```nmap -sC -sV -n -v 10.0.11.10``` - Nmap with script scan, port version mode, no DNS resolution, and verbose mode - High severity alert

In the alerts page of the web interface, you can view the SIEM's response to these commands.

![siemAlerts.jpg failed to load.]({{ site.baseurl }}/assets/images/homelab/buildPart9/siemAlerts.jpg)

## Summary
So we've installed security onion which sits on VLAN 15 and monitors traffic on VLAN 16. We configured DNS resolution to allow access to the web interface. Finally, we copied the traffic from VLAN 11 (corporate LAN) over to VLAN 16 in our firewall so that the SIEM can see it without monitoring VLAN 11 directly. 

