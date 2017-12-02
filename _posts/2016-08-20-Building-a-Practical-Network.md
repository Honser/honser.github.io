---
layout: post
title:  "Building a Practical Network"
date:   2016-08-20 19:00:00 -0600
categories: jekyll update
---
        | Cisco IOS Command |

>Source : <https://github.com/Honser/Building-a-Practical-Network>


--- 

#### Objectives
- Availability
: Using VLAN, faster transmission of traffic and load minimization for transmission equipment.
- Scalability  
: Assuring scalability with hierarchical structure.
- Dualization  
: When detecting a failure, ensure quick recovery with HSRP.
- Security  
: Guarantee of Confidentiality and integrity for data through filtering and intrusion dectection function.

&nbsp;

#### Components
- Equipments  
: Cisco Router 2621xm, Cisco Switch 3560(L3), Cisco Switch 3560(L2)  
- Internal  
: Basic config, Trunk, Etherchannel, VTP, VLAN, Portfast, RSTP, L3 Switch, SVI, HSRP, EIGRP, PVST, Inter-VLAN, NTP, DHCP Server
- External  
: OSPF, DMVPN, EZVPN
- Security  
: Syslog, AAA/ACS, CBAC, IPSec

&nbsp;

#### Diagram
![architecture]({{ site.baseurl }}/images/architecture.png)

&nbsp;

 
#### Project Documents
[netproject.pdf]({{ site.baseurl }}/images/netproject.pdf)

&nbsp;

