---
layout: single
title:  "Blue Team Homelab Part 5 - Corporate LAN Server"
date:   2024-10-02
categories: Homelab
---

For part 5 of making a homelab, we will be setting up a Windows Server on our corporate LAN network.

## Installing Windows Server
To start, we will be installing a server that will handle active directory, DNS, and file share. For this purpose, we will use Windows Server 2022. Ordinarily there would be a cost to gain access to a Windows Server license, but Microsoft offers an evalutation trial [here](https://www.microsoft.com/evalcenter/evaluate-windows-server-2022) for 180 days of free use.

The license key is embedded in the ISO file, which should work right out of the box without asking for a license. Some hypervisors, like vmware however, will create a floppy disk media by default, inside of which the Windows Server may look for the product key instead of looking inside of the ISO file. To fix this issue, ensure to delete any additional media aside from the ISO before booting the machine. Lastly, ensure that you download the Standard Desktop Experience version, unless you like working strictly from the command line.

## Configuration
### Network and Hostname

First we will set the VM network adapter to VLAN 11 and configure a static IP address.

![winServConf.jpg failed to load]({{ site.baseurl }}/assets/images/homelab/winServConf.jpg)

I will also change hostname of the computer in settings, which will trigger a restart.

![winServHostname.jpg failed to load]({{ site.baseurl }}/assets/images/homelab/winServHostname.jpg)


### Roles and Features
In the server manager application, go to ```Manage > Add Roles and Features```. Accept the default settings until *Server Roles*. Once you get to *Server Roles*, select the following:
* Active Directory Domain Service
* DHCP Server
* DNS Server
* File and Storage Services
* Remote Access

Continue the process with default settings and install at the end. Once the install completes, there should be notifications for some reuqired configurations under the flag icon at the top. 

![serverManagerPostInstall.jpg failed to load]({{ site.baseurl }}/assets/images/homelab/serverManagerPostInstall.jpg)

Start by hitting *Promote this server to a domain controller*.

Add a new forest and enter the domain you've been using.

![ADDomain.jpg failed to load]({{ site.baseurl }}/assets/images/homelab/ADDomain.jpg)

Enter the login credentials for the Administrator user and continue with default settings until the end.

![ADDomain2.jpg failed to load]({{ site.baseurl }}/assets/images/homelab/ADDomain2.jpg)

After installation, the computer will restart automatically and you should see your domain on the login screen now.

![winServLogin.jpg failed to load]({{ site.baseurl }}/assets/images/homelab/winServLogin.jpg)

### DHCP
Now we will configure our DHCP server. In the server manager application, start by going to ```Tools > DHCP```, right click *IPv4* and select *New Scope*.

![winServDHCP.jpg failed to load]({{ site.baseurl }}/assets/images/homelab/winServDHCP.jpg)

Now enter the name for the network with a description.

![winServDHCP2.jpg failed to load]({{ site.baseurl }}/assets/images/homelab/winServDHCP2.jpg)

Next, for the range of IP addresses that DHCP can give out, I opted to use *10.0.11.51 - 10.0.11.150* with a 24 bit subnet mask. For the next 3 pages (Exclusions, Lease Duration, and Configure DHCP options), I will use the default values. I don't want to exlclude any IPs, the default 8 days for a DHCP lease is ok, and we will configure DHCP options now. Next I will set the default gateway address to the IP address of the domain controller.

![winServDHCP3.jpg failed to load]({{ site.baseurl }}/assets/images/homelab/winServDHCP3.jpg)

For *Domain Name and DNS Servers*, the default domain and IP address should match what you set earlier and no change should be needed. For *WINS Servers*, I will add the windows server IP address with no server name. For the rest of this wizard, default settings are good. 

Now you should see all of the information you filled out under the DHCP window.

![winServDHCP4.jpg failed to load]({{ site.baseurl }}/assets/images/homelab/winServDHCP4.jpg)


### DNS
First we are going to set up a reverse lookup zone. This basically means DNS will be given an IP address and return a domain name. In the server manager application, go to ```Tools > DNS```. Right click on *Reverse Lookup Zones* and create a new zone. Go with all default settings until prompted to enter an IP address, in which you will enter the network portion of your corporate lan IP. In my case, that is ```10.0.11```. Continue with defaults until completed.

![reverseLookupZone.jpg failed to load]({{ site.baseurl }}/assets/images/homelab/reverseLookupZone.jpg)

Next I will add a DNS entry for the firewall by right clicking on the domain name under *Forward Lookup Zones* and selecting *New Host*. 

![newDNSEntry.jpg failed to load]({{ site.baseurl }}/assets/images/homelab/newDNSEntry.jpg)

Enter a name and the corresponding IP address that points to the firewall. Also check *Create Associated Pointer (PTR Record)*. A PTR record is the exact opposite of an A record, meaning that it takes an IP address and returns a domain name.

![newDNSEntry2.jpg failed to load]({{ site.baseurl }}/assets/images/homelab/newDNSEntry2.jpg)

A quick nslookup command lets us check that these changes are working.

![nslookupWinServ.jpg failed to load]({{ site.baseurl }}/assets/images/homelab/nslookupWinServ.jpg)

### Static Routing
Now, it's time to make this DC a routing point so that it can access the SIEM (which will be set up soon). In server manager, go to ```Tools > Routing and Remote Access```, right click the DC, and select *Configure and Enable Routing and Remote Access*. Continue through the wizard, and select *Custom* and then *LAN*.

Next, right click *IPv4* under the DC, and add a static route towards the IP address that you want to assign to the SIEM.

![remoteAccess.jpg failed to load]({{ site.baseurl }}/assets/images/homelab/remoteAccess.jpg)

### Users and Groups
For this next section, we will create a range of users. While some of these users will have good passwords, a few users will have weak passwords to simulate an insecure (and semi-realistic) security setup. 

First we will disable the password complexity requirement on the windows server. In server manager, go to ```Tools > Group Policy Management```. Then, navigate to *Default Domain Policy* and edit it.

![passwordSettings1.jpg failed to load]({{ site.baseurl }}/assets/images/homelab/passwordSettings1.jpg)

Next, navigate to ```Computer Configuration > Policies > Windows Settings > Security Settings > Account Policies > Password Policy```. And adjust settings as preferred, which in my case, I will disable password complexity and reduce minimum characters to 5. 

![passwordSettings2.jpg failed to load]({{ site.baseurl }}/assets/images/homelab/passwordSettings2.jpg)

To reiterate, this is being done to simulate a bad password policy... please don't ever do this in a live setting.

Now let's create our groups for our users. Go to ```Tools > Active Directory Users and Computers```, right click the name, and create a new organizational unit called *Groups*. Then right click *Groups* and create some groups with default settings. I will make *Marketing*, *HR*, and *IT*.

![groups.jpg failed to load]({{ site.baseurl }}/assets/images/homelab/groups.jpg)

Next let's create our users. On the same page under the domain, right click on *Users* and create a new user with whatever name you want. I will make some users with strong passwords, and some with bad common passwords from this [list](https://www.kaggle.com/datasets/wjburns/common-password-list-rockyoutxt). Set passwords to never expire and make sure you make at least 5 users. Also consider saving those passwords somewhere so you don't have to reset them when you test the user accounts.

![newUser.jpg failed to load]({{ site.baseurl }}/assets/images/homelab/newUser.jpg)

Now add each user to a group and make sure one user is also in the Administrator group. To add a user to a group, just right click a user, select *Add to a Group* and just type the group name in.

### Local Admin Policy
We will now create a local admin policy so that a local admin account can manage host devices. In Server manager, go to ```Tools > Group Policy Management```. Right click your domain and select *Create a GPO in this domain...* and name it Local Admin GPO.

![localAdmin1.jpg failed to load]({{ site.baseurl }}/assets/images/homelab/localAdmin1.jpg)

Now edit it and navigate to ```Computer Configuration > Policies > Windows Settings > Security Settings > Restricted Groups```. Create a new group here, which I will name *Local Administrators - Workstations* but choose whatever you would like. Add at least one user in here.

![localAdmin2.jpg failed to load]({{ site.baseurl }}/assets/images/homelab/localAdmin2.jpg)

### Shared Folder
The last step is to create a shared folder. This folder will support SMBv1, which is an insecure version of the SMB protocol that should be avoided. We will do this to test tools that should detect SMBv1 to avoid its use.

In the C: drive, create the following folder hierarchy, but with your own group names.
``` 
"C:"
|-- "sharedFolder"
    |-- "Marketing"
    |-- "IT"
    |-- "HR"
```

Now we will share these folders with the groups we made. Right click the *HR* folder and select *Properties*. Go to *Sharing* and add the HR group with read/write permissions. Now do this for the all the subfolders of *sharedFolder* with the corresponding groups.

![sharedFolder.jpg failed to load]({{ site.baseurl }}/assets/images/homelab/sharedFolder.jpg)

