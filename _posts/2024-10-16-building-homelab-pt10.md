---
layout: single
title:  "Building Blue Team Homelab Part 10 - HIDS Agents"
date:   2024-10-26
categories: Homelab
---

Now that the SIEM is listening to network traffic, we will install HIDS agents on our end devices to monitor individual computers as well. In order to do this, we will utilize a built-in tool called *Elastic Fleet*.

## Installing HIDS Agents
### Firewall Rules
First off, we will need to configure the firewall rules in security onion to allow our endpoint device agents to communicate back to our SIEM. In the SOC web interface, naviagate to ```Administration > Configuration > firewall > hostgroups > elastic_agent_endpoint```, and add the IPs of the devices that will have elastic agents on them. These will be the devices on our corporate LAN, so our windows server, the windows workstations, and our ubuntu server.

![elasticAgentEndpoint.jpg failed to load.]({{ site.baseurl }}/assets/images/homelab/buildPart10/elasticAgentEndpoint.jpg)

Additionally, we need to allow the corporate LAN subnet to send elastic agent logs to our security subnet in our pfSense firewall rules. Add a rule to allow TCP/UDP protocols from any source towards port 8220 on the IP of your SIEM.

![pfSenseElasticAgent.jpg failed to load.]({{ site.baseurl }}/assets/images/homelab/buildPart10/pfSenseElasticAgent.jpg)

### Linux Installation
I will start by installing the elastic agent on our ubuntu server. All of the elastic agent installations are located in the SOC web interface in the *Downloads* tab. Since this server does not have internet connectivity, I will install it on our DFIR VM and send it over via SFTP. 

![sftpLinux.jpg failed to load.]({{ site.baseurl }}/assets/images/homelab/buildPart10/sftpLinux.jpg)

Now, on our ubuntu server, lets make that file an executable and then execute it. If the firewall rules we applied earlier were successful, the installer log should indicate that the installation completed without termination. I will also move the elastic-agent folder one level up to clean up the directory a bit.

``` bash
# make executable and execute
chmod +x ./so-elastic-agent_linux_amd64
sudo ./so-elastic-agent_linux_amd64

# check log file to ensure the installation succeeded and could communicate to SIEM
cat SO-Elastic-Agent _Installer.log

# clean up directory after install
sudo mv so-elastic-agent_source/elastic-agent elastic-agent
rmdir so-elastic-agent_source

# cd into this directory before executing the final command
cd elastic-agent
```

Lastly, we will need to enroll this agent into our elastic fleet to actually view it. In the SOC web interface, open elastic fleet in the sidebar, and login using the same credentials. 

Select *Add agent*, choose the *so-grid-nodes_general* policy, and scroll down to part 3 where it shows some commands to execute. Copy the last line which should look like: ```sudo ./elastic-agent install --url=(urlHere) --enrollment-token=(longCode)```. Also, add a ```--insecure``` tag to this command, or else you will get an error looking for a certificate that we never set up.

``` bash
# once again, make sure you are in the elastic-agent directory when you run this command
sudo ./elastic-agent install --insecure --url=https://hlsiem.homelab.lan:8220 --enrollment-token=UE50WHdaSUJ4blphZUdJUFZ1cXk6OFJkanVMS19TbFdWN0N2QWI3RzlUdw==
```

Shortly after running this command, this agent should appear in the fleet list.

![ubuntuServerFleet.jpg failed to load.]({{ site.baseurl }}/assets/images/homelab/buildPart10/ubuntuServerFleet.jpg)

### Windows Installation
Luckily, the windows installation is significantly easier than the linux version. After sending the installer to the windows server, I just placed it in the IT shared folder so that both of the workstations can easily access it. All you need to is run the executable and you should see the agent has already joined the fleet shortly after. Unfortunately, elastic agents for Windows 7 are EOL, so that machine wont have endpoint monitoring.

Finally, all of our corporate LAN devices have an elastic agent installed on them for endpoint monitoring. We are basically done with building the homelab at this point, and all that's left to do is start tinkering with all the tools we have set up.
