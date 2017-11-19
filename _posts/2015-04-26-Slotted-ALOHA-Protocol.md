---
layout: post
title:  "Slotted ALOHA Protocol"
date:   2015-04-26 19:00:00 -0600
categories: jekyll update
---
	| C |

>Source : <http://www.github.com/honser>  


--- 
&nbsp;

#### Objectives
- Understand slotted ALOHA protocol
- Design the algorithm of the protocol in detail
- Simulate ALOHA protocol  
  
  
  

---
#### ALOHA?  

ALOHA is a system for coordinating and arbitrating access to a shared communication Networks channel. It was developed in the 1970s by Norman Abramson and his colleagues at the University of Hawaii. The original system used for ground based radio broadcasting, but the system has been implemented in satellite communication systems.

A shared communication system like ALOHA requires a method of handling collisions that occur when two or more systems attempt to transmit on the channel at the same time. In the ALOHA system, a node transmits whenever data is available to send. If another node transmits at the same time, a collision occurs, and the frames that were transmitted are lost. However, a node can listen to broadcasts on the medium, even its own, and determine whether the frames were transmitted. 

Slotted ALOHA is a refinement over the pure ALOHA. The Slotted ALOHA requires that time be segmented into slots of a fixed length exactly equal to the packet transmission time. Every packet transmitted must fit into one of these slots by beginning and ending in precise synchronisation with the slot segments. A packet arriving to be transmitted at any given station must be delayed until the beginning of the next slot. In contrast, for pure ALOHA, a packet transmission can begin at any time.  
&nbsp;

---
#### Pure ALOHA vs. Slotted ALOHA   
&nbsp;
![packet]({{ site.baseurl }}/images/6.png)
![ideal]({{ site.baseurl }}/images/8.png)

---

#### Simulation Results

The theoretical and simulated throughput of Slotted ALOHA are shown in the following graph.  
&nbsp;

![result]({{ site.baseurl }}/images/result.png)
