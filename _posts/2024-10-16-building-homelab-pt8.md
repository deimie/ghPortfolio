---
layout: single
title:  "Building Blue Team Homelab Part 8 - External Threats"
date:   2024-10-16
categories: Homelab
---

For this part, we will focus on setting up a Kali Linux machine which will sit on our simulated external network to serve as an attacker.

### Installation & Configuration
The installation of Kali Linux is found [here](https://www.kali.org/get-kali/), and it's nothing special compared to the last 7 installations we've already done, so I won't cover any in-depth steps.

Ensure the machine has internet with either the NAT or Bridged interface set on vmware. Then once base installation is complete, open the terminal and enter ```sudo apt update``` and ```sudo apt upgrade``` to get everything up to date.

Next, shut down the machine and set the interface to VLAN 10 or whatever your Corporate WAN network is. Reboot the machine, and next we will set a static IP. You can click the network icon at the top right and edit the network settings there.

![kaliNetConf.jpg failed to load.]({{ site.baseurl }}/assets/images/homelab/buildPart8/kaliNetConf.jpg)

Now our Kali machine is ready to attack our other machines from an internal network that our attacker somehow got access to.

### Port Forwarding
Next, lets enable some port forwarding settings on our pfSense firewall. Specifically, we want to enable external access to HTTP, SSH, and MariaDB on the web application from the last part.

Start the firewall, and in the web configurator, go to ```Firewall > NAT > Port Forwarding```. Then create 3 rules like the picture below shows. The first rule allows external connection to MariaDB, the second one allows connection to HTTP, and the last one allows connection to SSH. 

![pfPortForward.jpg failed to load.]({{ site.baseurl }}/assets/images/homelab/buildPart8/pfPortForward.jpg)

You may notice SSH has 4877 as a destination port, which is just a random number that I picked. This is an intentionally bad attempt at basically "hiding" the fact that SSH is publicly open, since you would need to connect from port 4877 instead of the typical 22. Here's a quick explanation of the settings to better understand why this works:

* Destination IP/Port - A public facing IP/port that will redirect to a private IP/port inside of the firewall 
* Redirect Target IP/Port - The actual IP/port that is inside of the firewall (in our case, the private IP of the Ubuntu server and the actual port we want to connect to)

So essentially, any machine trying to connect to SSH must use port 4877 on their side to establish a connection with port 22 on the Ubuntu Server. This does not actually increase security practically at all, as we will later find out.

### Test Connection
Now, a connection to the firewall should redirect to 10.0.11.10, the IP of the web application. Lets test it by entering 10.0.10.254 into our browser on Kali. 

![kaliDVWA.jpg failed to load.]({{ site.baseurl }}/assets/images/homelab/buildPart8/kaliDVWA.jpg)

Next, lets test out SSH. First off, lets make sure our Ubuntu Server is actually listening by enabling SSH. Enter ```systemctl start ssh```. We can check that the server is listening using ```netstat -lt```. 

![netstatSSH.jpg failed to load.]({{ site.baseurl }}/assets/images/homelab/buildPart8/netstatSSH.jpg)

Finally, I will connect over SSH using the following command ```ssh username@10.0.10.254 -p 4487```.

![kaliSSH.jpg failed to load.]({{ site.baseurl }}/assets/images/homelab/buildPart8/kaliSSH.jpg)
