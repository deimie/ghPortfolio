---
layout: single
title:  "Building Blue Team Homelab Part 7 - Web Server"
date:   2024-10-14
categories: Homelab
---

Now that the Windows server and workstations are configured, the last thing we will do on the Corporate LAN is configure a highly vulnerable web-server.

## Ubuntu Server VM
### Installation
To host our webserver, we will use [Ubuntu Server 22.04 LTS](https://ubuntu.com/download/server), which will be hosting the [Damn Vulnerable Web Application (DVWA)](https://github.com/digininja/DVWA) on it.

I will load mine with 2GB of RAM, 2 processor cores, and 40GB of storage. The installation is fairly straightforward, just ensure it has internet connectivity during installation. It should look like this when completed:

![ubuntuServer.jpg failed to load.]({{ site.baseurl}}/assets/images/homelab/buildPart7/ubuntuServer.jpg)

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

![ubuntuServNetplan.jpg failed to load.]({{ site.baseurl}}/assets/images/homelab/buildPart7/ubuntuServNetplan.jpg)

Lastly, it's always a good idea to create a snapshot before continuing, just in case anything breaks.

### Starting the Webserver
Now it's time to start the webserver. To do this, we will start the Apache service using the command ```systemctl start apache2```.

![startApache.jpg failed to load.]({{ site.baseurl}}/assets/images/homelab/buildPart7/startApache.jpg)

To view the web application, open a browser and enter ```http://10.0.11.10/DVWA/```. The default username is *admin* and the password is *password*.

### Database Configuration
At the bottom of the page, choose *Create / Reset Database*, then login with the default credentials.

Now we will make this database even less secure! Navigate to ```/etc/mysql/mariadb.conf.d``` and edit ```50-server.cnf```. Change the bind address from *127.0.0.1* to *0.0.0.0*, which will allow remote connections to the database.

``` bash
# bind-address = 127.0.0.1
bind-address = 0.0.0.0
```

Restart MariaDB (the database service) using ```sudo systemctl restart mariadb``` and check that your changes worked using ```ss -tnlp```.

![mariadbBindAddress.jpg failed to load.]({{ site.baseurl}}/assets/images/homelab/buildPart7/mariadbBindAddress.jpg)

I will also change the overall security level of the web application by changing the setting in under *DVWA Security*. The default is *impossible* but I will choose *low*.

![dvwaSecLevel.jpg failed to load.]({{ site.baseurl}}/assets/images/homelab/buildPart7/dvwaSecLevel.jpg)

### Apache Configuration
Ok, so now we will set up DNS for the web application. First, we will copy the apache default config file and edit it.

``` bash
cd /etc/apache2/sites-available
sudo cp 000-default.conf system.homelab.lan.conf
sudo nano system.homelab.lan.conf
```
We will edit this config file to have the domain we want, the admin account we want, and set the root folder correctly.

![apacheConf.jpg failed to load.]({{ site.baseurl}}/assets/images/homelab/buildPart7/apacheConf.jpg)

Next we will tell the server to use the new configuration, disable the old configuration, and then restart apache to apply the changes.

``` bash
sudo a2ensite system.homelab.lan.conf
sudo a2dissite 000-default.conf
systemctl restart apache2
```

Now edit ```/etc/hosts``` and add your domain to the first line so it looks like the following.

``` bash
127.0.0.1 localhost system.homelab.lan
```

Lastly, I will edit my host machine's hosts file. My host machine is running Windows, so the location of the host file is ```C:\Windows\System32\drivers\etc\hosts```. I will add an entry for the web application and for the Windows Server firewall.

``` bash
10.0.11.10 system.homelab.lan
10.0.11.5 firewall.homelab.lan
```

Now the host machine can open the web application and the firewall using domain names instead of IP addresses.

## Windows Server VM
### Active Directory Configuration
Now we will add a DNS record on our Corporate LAN windows server so that the VM workstations can access the web application using the domain name as well. Add a new A record in the server manager application under ```Tools > DNS```.

![winServDVWA.jpg failed to load.]({{ site.baseurl}}/assets/images/homelab/buildPart7/winServDVWA.jpg)

Now the damn vulnerable web application is accessible using a domain name on both the host machine and on the VM workstations in the Corporate LAN.