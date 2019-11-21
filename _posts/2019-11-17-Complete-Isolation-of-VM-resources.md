---
layout: post
title:  "Complete Isolation of VM resources"
description: ""
categories: articles
tags: [OpenStack, CPU, Isolation]
---

When we allocate CPUs to a guest for using them only for an application on it, we need to keep in mind that many other processes from host to guest can steal that CPUs and sometimes VM should compete for it. Not to let the host or other VMs interrupt, some configuration must be set for the resources to be dedicated completely. In our evironment, hyper-threading is enabled unless it's a special case.
<br>

![vm_architecture]({{ site.baseurl }}/images/isolation1.png#center)

<br>
## Kernel Parameters
Kernel parameters should be set to isolate CPUs from the general scheduler and also prevent them from handling interrupts. 

If our server has 2 sockets and 20 cpus(40 with Hyperthreading eanbled), 
<pre>
        Socket 0        Socket 1        
        --------        --------        
<b>Core 0  [0, 20]         [10, 30] <-- For OS and general purposes</b>
Core 1  [1, 21]         [11, 31]         
Core 2  [2, 22]         [12, 32]        
Core 3  [3, 23]         [13, 33]        
Core 4  [4, 24]         [14, 34]        
Core 5  [5, 25]         [15, 35]        
Core 6  [6, 26]         [16, 36]        
Core 7  [7, 27]         [17, 37] 
Core 8  [8, 28]         [18, 38] 
Core 9  [9, 29]         [19, 39] 
# CPU numbers might be different depending on the host OS.
</pre>
We can choose every core for VM except for the cores correspond to Core 0 on both NUMA nodes. edit **/etc/default/grub.cfg** file and add parameters to the GRUB_CMDLINE_LINUX line:
<pre>
GRUB_CMDLINE_LINUX="isolcpus=1-9,11-19,21-29,31-39 nohz_full=1-9,11-19,21-29,31-39 rcu_nocbs=1-9,11-19,21-29,31-39"
</pre>
Simply, ```isolcpus``` is to isolate the masked CPUs from the scheduler, ```nohz_full``` is not to let them receive timer ticks and ```rcu_nocbs``` is used to no schedule RCU on the CPUs.

**Note**:  
Even with ```nohz_full``` set, you might see some interrupt counts are still increasing on isolated cores.

After correctly editting the grub file, apply the change to the grub.cfg by running:
<pre>
#grub2-mkconfig -o /boot/grub2/grub.cfg
</pre>

Reboot the system and check the grub command line with ```cat /proc/cmdline```.

Above step is for host and guest both.
<br><br>

## IRQBalance
the irqbalance is a linux daemon that distributes interrupts across CPUs in your system. Most of linux distributions enable irqbalance by default and it is recommended not to disable it without a very good reason because it causes all the interrupts handled by CPU0. The problem is that irqbalance might not avoid IRQs on the isolated cores. There is a ```IRQBALANCE_BANNED_CPUS``` parameter to prevent it or you can manually assign IRQs to specific cores after disabling irqbalance daemon.

**Note**:  
from RHEL7.2, automatically reads ```isolcpus``` parameter and avoids it. You had better check documents for the other linux distributions. By the way, ```IRQBALANCE_BANNED_CPUS``` might not work.
<br>
<br>
## vCPU Pool
We can define which physical CPUs can be used for instance vCPU. On each compute node, open ```/etc/nova/nova,conf``` and append following parameter with the range of cores:
<pre>
vcpu_pin_set=1-9,11-19,21-29,31-39
</pre> 

Once the change have been made, restart the ```nova-compute``` service.
<br>
<br>
## Overcommitment
OpenStack allows overcommiting CPU and RAM by default. Overcommitment increases the number of instances you can run, but reduces instance performance. It is obvious that it be beneficial for efficiency if performance is not a big deal. In this post, however, we shouldn't let it happen since our purpose is to keep our CPUs safe from others. By default, overcommitment of resources as follows:
* **vCPU** : 16 x physical cores
* **vMemory** : 1.5 x physical memories  

Edit ```/etc/nova/nova.conf``` on each compute node again and set ```cpu_allocation_ratio``` and ```memory_allocation_ratio``` to ```1.0```. Restarting the service is also required.
<br>
<br>
## CPU Pinning Policies
If we don't specify any information about CPU in flavor. By default, instance vCPUs aren't bound to particular pCPUs but they will float across pCPU like any other process. To pin vCPUs, CPU policies should be set to flavor extra-spec or image properties.  
<pre>
hw:cpu_policy=<b>dedicated</b>
hw:cpu_threads_policy=<b>prefer</b>
</pre>
vCPU can be bound to pCPU with ```dedicated```, and ```prefer(default)``` is for non-HT and HT both. Allocating vCPUs on thread siblings is possible with that.

**Note**:  
1.Even if all the pinned instances respect the policy, unpinned instances don't. They can use CPUs pinned instances have. **Host aggregates** should be used to isolate them completely.   
2.```cpu_threads_policy=isolate``` is to use one thread sibling only on each core like non-hyperthreading system.
<br>
<br>
## Measure CPU Utilization of Isolated Cores

### top
There are still possibilies our isolated, we believe, resources can be stolen. It can be host OS, Other instances, or guest OS. Remember that our goal is to use the core for dedicated workloads only not general purpose. One of the easiest way to check if it's being used by others is monitoring ```top``` command.  
<br>
![top_cmd_result]({{ site.baseurl }}/images/isolation2.png#center)
<br>
```%st``` field is a steal time, which is the percentage of time a vCPU waits for pCPU while the hypervisor is serving another vCPU from others or host itself. It might not be observed now depends on current workloads. Anyway, isolation has't been done well if the time is being stolen.


### Interrupts
On host and guest both, ```/proc/interrupts``` is useful to see which interrupts are assigned to the isolated cores.  
<br>
![interrupts]({{ site.baseurl }}/images/isolation3.png#center)

