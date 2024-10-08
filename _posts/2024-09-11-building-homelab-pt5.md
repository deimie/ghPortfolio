---
layout: single
title:  "Blue Team Homelab Part 5 - Corporate LAN"
date:   2024-10-02
categories: Homelab
---

Lorem ipsum description

## Installing Windows Server
To start, we will be installing a server that will handle active directory, DNS, and file share. For this purpose, we will use Windows Server 2022. Ordinarily there would be a cost to gain access to a Windows Server license, but Microsoft offers an evalutation trial [here](https://www.microsoft.com/evalcenter/evaluate-windows-server-2022) for 180 days of free use.

The license key is embedded in the ISO file, which should work right out of the box without asking for a license. Some hypervisors, like vmware however, will create a floppy disk media by default, inside of which the Windows Server may look for the product key instead of looking inside of the ISO file. To fix this issue, ensure to delete any additional media aside from the ISO before booting the machine. Lastly, ensure that you download the Standard Desktop Experience version, unless you like working strictly from the command line.

## Configuration


![winServConf.jpg failed to load]({{ site.baseurl }}/assets/images/homelab/winServConf.jpg)

### Roles and Features

![serverManagerPostInstall.jpg failed to load]({{ site.baseurl }}/assets/images/homelab/serverManagerPostInstall.jpg)

![ADDomain.jpg failed to load]({{ site.baseurl }}/assets/images/homelab/ADDomain.jpg)

![ADDomain2.jpg failed to load]({{ site.baseurl }}/assets/images/homelab/ADDomain2.jpg)

![winServLogin.jpg failed to load]({{ site.baseurl }}/assets/images/homelab/winServLogin.jpg)

### DHCP
Now we will configure our DHCP server. In the server manager application, start by going to ```Tools > DHCP```, right click *IPv4* and select *New Scope*.

![winServDHCP.jpg failed to load]({{ site.baseurl }}/assets/images/homelab/winServDHCP.jpg)

Now enter the name for the network with a description.

![winServDHCP2.jpg failed to load]({{ site.baseurl }}/assets/images/homelab/winServDHCP2.jpg)

Next, for the range of IP addresses that DHCP can give out, I opted to use *10.0.11.51 - 10.0.11.150* with a 24 bit subnet mask. For the next 3 pages (Exclusions, Lease Duration, and Configure DHCP options), I will use the default values. I don't want to exlclude any IPs, the default 8 days for a DHCP lease is ok, and we will configure DHCP options now. Next I will set the default gateway address to the IP address of the windows server.

![winServDHCP3.jpg failed to load]({{ site.baseurl }}/assets/images/homelab/winServDHCP3.jpg)

For *Domain Name and DNS Servers*, the default domain and IP address should match what you set earlier and no change should be needed. For *WINS Servers*, I will add the windows server IP address with no server name. For the rest of this wizard, default settings are good. 

Now you should see all of the information you filled out under the DHCP window.

![winServDHCP4.jpg failed to load]({{ site.baseurl }}/assets/images/homelab/winServDHCP4.jpg)


### DNS
