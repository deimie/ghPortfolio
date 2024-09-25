---
layout: single
title:  "Blue Team Homelab Part 4 - Malware Analysis"
date:   2024-09-24
categories: Homelab
---

For part 4 of building this homelab environment, we will focus on setting up our malware anaylsis device. 

When it comes to building our isolated network for malware anaylsis, it is extremely important that we can analyze malicious files without any risk of the malware spreading to other virtual machines or your host machine outside of the contained VLAN. Oftentimes, malware will also try and establish a connection with other IPs over the internet, which is why it is so important that the malware is only allowed to run in an environment that is completed isolated from public internet connectivity.

### Malware Analysis VM
To handle malware anaylsis, we will be using Windows 10 with an MA distribution called [FlareVM](https://github.com/mandiant/flare-vm) installed on top of it. You can use Microsoft's [media creation tool](https://www.microsoft.com/en-us/software-download/windows10) to create a disk image of Windows, even if you don't have a valid license. Without a license, some personalization settings will be disabled, but this is not important for our use-case. 

During the creation, ensure that the windows machine has at least 80GB of storage. Once you have downloaded FlareVM from github, it is highly advised that you create a snapshot of Windows because any errors during the long installation process of FlareVM will likely leave you with a broken version of Windows, in which case simply reverting back the snapshot will save you lots of time.

Next, you will need to disable windows defender, otherwise it will prevent the FlareVM installation from running the necessary scripts to create and delete certain files. Windows defender is very persistent and difficult to fully disable, so I would recommend running this [script](https://github.com/ionuttbara/windows-defender-remover) in powershell as an administrator to remove it. The format ```.\fileName``` is used to execute a file in powershell. 

With windows defender out of the way, we can begin the installation process. With powershell open as an administrator, we will execute: ```Set-ExecutionPolicy unrestricted``` to escalate the permissions for running scripts. When prompted, respond with Y. Next, cd to your flareVM folder and run ```.\install.ps1``` to begin the installation. 

![powerShellFlareVM.jpg failed to load]({{ site.baseurl }}/assets/images/homelab/powerShellFlareVM.jpg)

The installation process is quite long and can take up to a few hours to fully complete. Once it is completed, you will be presented with a desktop environment that should resemble the following:

![flareVMDesktop.jpg failed to load]({{ site.baseurl }}/assets/images/homelab/flareVMDesktop.jpg)

### Network Configuration
Now that the installation is complete, we can change the VM interface to our isolated VLAN 19 and configure a static IP address for the machine.
> Side note: if you run ```ipconfig``` in command prompt and see an "autoconfiguration IPv4 address" being assigned instead of the static one you set, you likely chose a static IP that was already reserved for another device.

![flareVMNetwork.jpg failed to load]({{ site.baseurl }}/assets/images/homelab/flareVMNetwork.jpg)

The last step before this machine is ready to perform MA is to find a way to safely send malware to this machine. If this machine had internet connectivity you could just download malicous files online, but this is generally a bad idea. Instead, we will open an SSH port so that our DFIR machine can send malicous files to this one using a tool called [WinSCP](https://winscp.net/eng/index.php).

### Downloads Used for This Part
* [Download FlareVM](https://github.com/mandiant/flare-vm)
* [Download Windows](https://www.microsoft.com/en-us/software-download/windows10)
* [Remove Windows Defender](https://github.com/ionuttbara/windows-defender-remover)