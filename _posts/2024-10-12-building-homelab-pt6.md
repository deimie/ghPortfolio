---
layout: single
title:  "Blue Team Homelab Part 6 - Corporate LAN Workstations"
date:   2024-10-12
categories: Homelab
---

Now that we have set up the windows server, we will begin creating some workstation devices that we will connect to the server's domain.

## Windows 7 VM
First we will [install Windows 7](https://files.rg-adguard.net/file/bfa486e2-9395-5653-c3c4-e12ef71f6840), which is moderately more difficult to do now that Microsoft no longer supports this fossil. Luckily there are some sites like the one I linked that have some old archives of Windows 7 though. Even though Windows 7 is outdated, many companies still use legacy software and operating systems, so it's good to learn how to use them.

Installation is straightforward. Install the OS and change the interface to VMnet 11. 

### Firewall
By default, ICMP is blocked on windows, so let's unblock it so that we can ping this machine to verify that it is networking correctly. Go to ```Control Panel > System and Security > Windows Firewall``` then open *Advanced Settings* and go to *Inbound Rules*. Then find *File and Printer Sharing (Echo Request - ICMPv4-In)* and enable both entries, for domain and for private.

![win7Firewall.jpg failed to load.]({{ site.baseurl}}/assets/images/homelab/win7Firewall.jpg)

### Hostname and Joining Domain
Before we change the hostname and join the domain, we must ensure this device has an IP that can talk to the windows server. By default, this machine should immediately detect the Windows Server and receive an IP from the DHCP server, however this did not happen in my case. A simple fix for now is to temporarily give this machine a static IP so that it can talk to the Windows Server and join the domain. 

To do this go to ```Control Panel > Network and Internet > Network and Sharing Center``` and select *Change adapter settings*. Select the network adapter, hit *Properties*, then find the IPv4 entry and select *Properties* again. Ensure that the DNS server is the windows server IP, not the firewall IP. Restart the computer to apply these changes.

![win7staticIP.jpg failed to load.]({{ site.baseurl}}/assets/images/homelab/win7staticIP.jpg)

Next, to change the hostname, go to ```Control Panel > System and Security > System```. Select *Change Settings* and hit *Change* on the window that appears. Set the computer name to something realistic, which in this case I will name it *Meeting-PC01* to represent a PC that is used for joining meetings. Finally, make the computer a member of your domain. If 

![win7hostname.jpg failed to load.]({{ site.baseurl}}/assets/images/homelab/win7hostname.jpg)


### Testing the Domain User
Now you should see the domain on the login page. Select *Switch User* and log in with one of your HR user's credentials. Now let's test the shared folder by going into the network section in file explorer. You may be prompted to enable file sharing discovery at the top when you first open this page, which you will need to log in with the Administrator account to enable. If all is done correctly, udner the WIN-DC01 computer, you should see the shared folder with only the HR folder inside of it.

![win7SharedFolder.jpg failed to load.]({{ site.baseurl}}/assets/images/homelab/win7SharedFolder.jpg)

## Windows 10 VM
Now there will also be a Windows 10 VM workstation connected to our server. The great thing is that the steps to configure this VM are practically identical to what we just did on Windows 7, so I will just briefly mention a few things. 

Firstly, this time around, the server was automatically detected without doing anything, but the static IP solution I used for the Windows 7 machine will work if this doesn't happen. For the hostname, I will name it *IT-PC01* since this will be an IT workstation. Every other step of configuration was practically identical.

### Testing the Domain User
This time, I will login using a member of the IT group. Once logged in, I will just check the shared folder, and this time around, I should only have access to view the IT folder.