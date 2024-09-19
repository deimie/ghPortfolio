---
layout: single
title:  "Blue Team Homelab Part 1 - Network Topology"
date:   2024-09-02
categories: Homelab
---

Part 1 of many in setting up my own blueteam homelab for learning purposes. Part 1 will be setting up some of the core infrastructure and documenting the network topology.

---

### About this homelab
I am pursuing this project in an ongoing effort to learn more about networking and cybersecurity so that I can develop hands-on skills and familarize myself with industry tools. This homelab is largely based off of [this](https://facyber.me/posts/blue-team-lab-guide-part-1/) project by facyber, which I am using to expand my knowledge of tools and technologies within this industry.

---

### Network Topology
**Virtual Network Editor in vmware**\
To start off, I created 5 additional network interfaces in the Virtual Network Editor. These interfaces should be host-only with DHCP disabled. VMnet1 is the exception, which will be host-only with DHCP enabled. I also set each subnet address to match their VLAN #s for simplicity, but this is not required. 

VMnet0 is a bridged interface, meaning that virtual machines will use the host computer’s ethernet adapter to access a network. It is presently set to automatically detect an ethernet adapter from the host machine, but it can be manually set to the proper ethernet adapter if issues arise. VMnet8 is our NAT interface to allow communication to the public internet. Both of these are defaults and should remain untouched.

![networkTop.jpg]({{ site.baseurl }}/assets/images/homelab/networkTop.jpg)

| VLAN    | Function in simulated environment                                                                                                                                    |
|---------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| VLAN 1  | Network management                                                                                                                                                   |
| VLAN 10 | Corporate WAN VLAN. Only has access to corporate VLAN 11                                                                                                             |
| VLAN 11 | Corporate LAN network. Servers and end devices will be kept here. Only has access to fake internet network.                                                          |
| VLAN 15 | Security VLAN with VMs for DFIR and OSINT. Simulates a security team with network permission to access the corporate network. Will have access to the real internet. |
| VLAN 19 | Isolated LAN network with VMs for malware analysis.                                                                                                                  |


**Subnets and Gateways**\
Documenting which subnets and gateways correspond to which VLAN will be an extremely useful reference throughout this project.

| Name          | Domain     | VLAN     | Subnet       | Gateway     |
|---------------|------------|----------|--------------|-------------|
| Management    | N/A        | 1        | 10.0.1.0/24  | 10.0.1.1    |
| Corporate WAN | N/A        | 10       | 10.0.10.0/24 | 10.0.10.254 |
| Corporate LAN | home.arpa  | 11       | 10.0.11.0/24 | 10.0.11.254 |
| Security      | home.arpa  | 15       | 10.0.15.0/24 | 10.0.15.254 |
| Isolated LAN  | N/A        | 19       | 10.0.19.0/24 | 10.0.19.254 |
