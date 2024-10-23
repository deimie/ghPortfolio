---
layout: single
title:  "Building Blue Team Homelab Part 9 - SIEM Installation"
date:   2024-10-20
categories: Homelab
---

For this part of building the homelab, we will be installing a SIEM on our security VLAN.

## Security Onion
### Installation
For our SIEM, we will be using an open-source software called [Security Onion](https://github.com/Security-Onion-Solutions/securityonion/blob/2.4/main/DOWNLOAD_AND_VERIFY_ISO.md). Since we will ultimately be using the Standalone version of Security Onion, the bare minimum specs are: 200 GB of storage, 4 CPU cores, and 16 GB of RAM.

Additionally, we will need to create one more VLAN in vmware since this SIEM will need access to 2 NICs. The first NIC will be our security VLAN 15, and the second one I will designate to a new VLAN 16. The second NIC will be explained later.

![newVLAN16.jpg failed to load.]({{ site.baseurl }}/assets/images/homelab/buildPart9/newVLAN16.jpg)

This is roughly how your box settings should look. I'm using 18GB of RAM to slightly improve performance since 16GB is the bare minimum.

![onionSpecs.jpg failed to load.]({{ site.baseurl }}/assets/images/homelab/buildPart9/onionSpecs.jpg)

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

