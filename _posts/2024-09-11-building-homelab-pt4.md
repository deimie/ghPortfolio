---
layout: single
title:  "Blue Team Homelab Part 4 - Malware Analysis"
date:   2024-09-24
categories: Homelab
---

For part 4 of building this homelab environment, we will focus on setting up our malware anaylsis device. 

When it comes to building our isolated network for malware anaylsis, it is extremely important that we can analyze malicious files without any risk of the malware spreading to other virtual machines or your host machine outside of the contained VLAN. Oftentimes, malware will also try and establish a connection with other IPs over the internet, which is why it is so important that the malware is only allowed to run in an environment that is completed isolated from public internet connectivity.

## Windows MA
To handle malware anaylsis, we will be using Windows 10 with an MA distribution called [FlareVM](https://github.com/mandiant/flare-vm) installed on top of it. You can use Microsoft's [media creation tool](https://www.microsoft.com/en-us/software-download/windows10) to create a disk image of Windows, even if you don't have a valid license. Without a license, some personalization settings will be disabled, but this is not important for our use-case. 

During the creation, ensure that the windows machine has at least 80GB of storage. Once you have downloaded FlareVM from github, it is highly advised that you create a snapshot of Windows because any errors during the long installation process of FlareVM will likely leave you with a broken version of Windows, in which case simply reverting back the snapshot will save you lots of time.

Next, you will need to disable windows defender, otherwise it will prevent the FlareVM installation from running the necessary scripts to create and delete certain files. Windows defender is very persistent and difficult to fully disable, so I would recommend running this [script](https://github.com/ionuttbara/windows-defender-remover) in powershell as an administrator to remove it. The format ```.\fileName``` is used to execute a file in powershell. 

With windows defender out of the way, we can begin the installation process. With powershell open as an administrator, we will execute: ```Set-ExecutionPolicy unrestricted``` to escalate the permissions for running scripts. When prompted, respond with Y. Next, cd to your flareVM folder and run\ ```.\install.ps1``` to begin the installation. 

![powerShellFlareVM.jpg failed to load]({{ site.baseurl }}/assets/images/homelab/powerShellFlareVM.jpg)

The installation process is quite long and can take up to a few hours to fully complete. Once it is completed, you will be presented with a desktop environment that should resemble the following:

![flareVMDesktop.jpg failed to load]({{ site.baseurl }}/assets/images/homelab/flareVMDesktop.jpg)

### FlareVM Network Configuration
Now that the install is complete, we will change the VM interface to our isolated VLAN 19 and configure a static IP address for the machine.

> Side note: if you run ```ipconfig``` in command prompt and see an "autoconfiguration IPv4 address" being assigned instead of the static one you set, you likely chose a static IP that was already reserved for another device.

![flareVMNetwork.jpg failed to load]({{ site.baseurl }}/assets/images/homelab/flareVMNetwork.jpg)


## Linux MA
Next, we will set up our Linux VM to handle malware anaylsis. For this purpose, we will use [REMnux](https://remnux.org/). The installation of REMnux is pretty simple, and should be fairly quick as well since it is a lightweight OS. 

Once REMnux is installed, ensure that it is connected to the internet so that we can apply any updates using the commands ```sudo apt update``` and ```sudo apt upgrade -y```. We will also install OpenSSH on this machine using ```sudo apt-get install ssh``` and ```sudo apt-get install openssh-server```.  

### Remnux Network Configuration
Once all the commands have successfully ran, switch the interface to VLAN 19. Next, we will configure a static IP and a gateway for this machine. To do this, edit the file ```/etc/netplan/01-netcfg.yaml``` so that it has a static IP and the proper gateway, as well as your chosen domain name. You may need to add new lines for most of the fields seen below.

![remnuxNetConf.jpg failed to load]({{ site.baseurl }}/assets/images/homelab/remnuxNetConf.jpg)

Lastly, input ```sudo netplan apply``` to apply the changes from that file.

## SFTP Testing
Now that both machines are on the same VLAN, we can begin using SFTP to transfer files. Start with opening the SSH server on Remnux using ```service ssh start```. The default password for this machine is "malware". You may notice when running ```service ssh status``` that the server is listening on ```0.0.0.0```, which just means that it is listening using all of the IP addresses on the local machine, which in my case is only ```10.0.19.19```.

![openSSHRemnux.jpg failed to load]({{ site.baseurl }}/assets/images/homelab/openSSHRemnux.jpg)

Next, on FlareVM, connect to the SSH server using ```sftp remnux@10.0.19.19``` using the remnux hostname and IP address. Here is a little cheatsheet for some of the common commands to do a basic file transfer.

>```ls```: Lists the files and directories in the current remote directory.\
>```dir```: Lists the files and directories in the current local directory.\
>```cd [directory_name]```: Changes the current remote directory to directory_name.\
>```lcd [directory_name]```: Changes the current local directory.\
>```pwd```: List the current directory on the remote machine.\
>```lpwd```: List the current directory on the local machine.\
>```get [filename]```: Downloads file from the remote server to the local machine.\
>```put [filename]```: Uploads file from the local machine to the remote server.\
>```exit``` or ```quit```: Closes the connection to the remote server and exits SFTP.\

![sftpTest.jpg failed to load]({{ site.baseurl }}/assets/images/homelab/sftpTest.jpg)

In this example, I connected to the SSH server and listed the local and remote present working directories. Then I moved a file called tester.txt from FlareVM to Remnux, then downloaded a file called newFile.txt from Remnux to FlareVM.

Now both of our MA machines are completely set up and ready to transfer files between each other on their own isolated VLAN.


## Downloads Used for This Part
* [Download Windows](https://www.microsoft.com/en-us/software-download/windows10)
* [Remove Windows Defender](https://github.com/ionuttbara/windows-defender-remover)
* [Download FlareVM](https://github.com/mandiant/flare-vm)
* [Download REMnux](https://remnux.org/)