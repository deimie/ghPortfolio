---
layout: single
title:  "Blue Team Homelab Part 7 - Web Server"
date:   2024-10-14
categories: Homelab
---

Now that the Windows server and workstations are configured, the last thing we will do on the Corporate LAN is configure a highly vulnerable web-server.

## Ubuntu Server VM
### Installation
To host our webserver, we will use [Ubuntu Server 22.04 LTS](https://ubuntu.com/download/server), which will be hosting the [Damn Vulnerable Web Application (DVWA)](https://github.com/digininja/DVWA) on it.

I will load mine with 2GB of RAM, 2 processor cores, and 40GB of storage. The installation is fairly straightforward, just ensure it has internet connectivity during installation. It should look like this when completed:

![ubuntuServer.jpg failed to load.]({{ site.baseurl}}/assets/images/homelab/ubuntuServer.jpg)

Next, we will update the server and install the web application as well as some neccesary tools and some generally useful tools. ```apache2``` is the framework for the HTTP server we will host. Once Apache is installed, a new directory ```var/www/html``` should be created, which is where we will install the DVWA. There are a few ways to install the DVWA, but I will personally use the automated script provided in the README of the github project. 

``` bash
sudo apt update
sudo apt install -y apache2 mariadb-server mariadb-client php php-mysqli php-gd libapache2-mod-php fping nano
sudo wget https://raw.githubusercontent.com/IamCarron/DVWA-Script/main/Install-DVWA.sh
sudo chmod +x Install-DVWA.sh
sudo ./Install-DVWA.sh
```

### Network
Once these installations are complete, change the interface to VLAN 11. Now we will configure a static IP address by creating and editing this file: ```/etc/netplan/01-netcfg.yaml```. Configure as shown below, but your network adapter may or may not be *ens33*, so first check using ```ip a```. Finally, type ```sudo netplan apply``` to apply these changes.

![ubuntuServNetplan.jpg failed to load.]({{ site.baseurl}}/assets/images/homelab/ubuntuServNetplan.jpg)

Lastly, it's always a good idea to create a snapshot before continuing, just in case anything breaks.

### Starting the Webserver
Now it's time to start the webserver. To do this, we will start the Apache service using the command ```systemctl start apache2```.

![startApache.jpg failed to load.]({{ site.baseurl}}/assets/images/homelab/startApache.jpg)

To view the web application, open a browser and enter ```http://10.0.11.10/DVWA/```. The default username is *admin* and the password is *password*.

### Database Configuration
At the bottom of the page, choose *Create / Setup Database*, then login with the default credentials.