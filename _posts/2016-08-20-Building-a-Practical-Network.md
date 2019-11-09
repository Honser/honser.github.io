---
layout: post
title:  "Building a Practical Network"
category: articles
tags: [Network, Cisco, Project]
--- 

>Source : <https://github.com/Honser/Building-a-Practical-Network>

--- 


## Objectives  

- Availability
: Using VLAN, faster transmission of traffic and load minimization for transmission equipment.
- Scalability  
: Assuring scalability with hierarchical structure.
- Dualization  
: When detecting a failure, ensure quick recovery with HSRP.
- Security  
: Guarantee of Confidentiality and integrity for data through filtering and intrusion dectection function.
<br>
<br>

## Components

- Equipments  
: Cisco Router 2621xm, Cisco Switch 3560(L3), Cisco Switch 3560(L2)  
- Internal  
: Basic config, Trunk, Etherchannel, VTP, VLAN, Portfast, RSTP, L3 Switch, SVI, HSRP, EIGRP, PVST, Inter-VLAN, NTP, DHCP Server
- External  
: OSPF, DMVPN, EZVPN
- Security  
: Syslog, AAA/ACS, CBAC, IPSec
<br>
<br>

## Diagram

![architecture]({{ site.baseurl }}/images/architecture.png)
<br>
<br>
 
## Project Documents

[netproject.pdf]({{ site.baseurl }}/images/netproject.pdf)

