---
layout: post
title:  "Provisioning a VM with SR-IOV interfaces"
description: "How to make and use SR-IOV interfaces"
categories: articles
tags: [OpenStack, SRIOV]
---

As most of our VNFs need multiple interfaces from limited physical ports and also require near line-rate performance, SR-IOV fits our needs so well.

<br>

## SR-IOV?
Single Root I/O Virtualization(SR-IOV) is a technology, by Intel created to improve the networking performance of virtual machines, and which is a specification that allows a PCIe device to appear to be multiple seperate physical PCe devices. SR-IOV simply consists of physical function(PF) and virtual function(VF). PF is a physical PCIe device has full PCIe device features; VF is a kind of lightweight PCIe and it can only support data transmission. Although the maximum number of VFs per port depends on the device, 64 or 32 VFs seems to be the upper limit for most devices. 


![sriov_overview]({{ site.baseurl }}/images/sriov_overview2.png)

<br>

## Requirements
- Hardware, OS, and firmware(BIOS or UEFI) must support SR-IOV.
- Make sure Intel VT-d(or AMD-Vi) features is enabled(IOMMU is generic name for VT-d and AMD-Vi)

**Note**:
VT-d stands for Intel Virtualization Technology for Directed I/O and should not be confused with VT-x Intel Virtualization Technology. VT-x allows one hardware platform to function as multiple “virtual” platforms while VT-d improves security and reliability of the systems and also improves performance of I/O devices in virtualized environments.

**Note**:
SR-IOV requires support from the operating system and hardware platform. Some systems support SR-IOV through some PCIe slots, but not on some other slots. In my case, slot 3 of DL380p Gen8 server.

<br>

## IOMMU
I/O Memory Management Unit(IOMMU) is not enabled by default in most distribution of Linux. IOMMU is required to assign VFs to any VM. To enable it, configure `intel_iommu=on` in the grub file and add `iommu=pt` to get the better performance. After edit the file, execute below command and reboot the server for the change to take effect.

{% highlight properties %}

#grub2-mkconfig -o /boot/grub2/grub.cfg

{% endhighlight %}

To check if IOMMU is enabled,

{% highlight properties %}

$dmesg | grep -e DMAR -e IOMMU
...
DMAR: IOMMU enabled
DMAR: Intel(R) Virtualization Technology for Directed I/O
...

{% endhighlight %}

The output should display that **IOMMU**, **Directed I/O**, or **Interrupt Remapping** is enabled like above.

It's also needed to check the devices we want to passthrough are in a seperate IOMMU group and this can be verified with:

{% highlight properties %}

$find /sys/kernel/iommu_groups/ -type l

{% endhighlight %}

An IOMMU group is the smallest set of physical devices that can be passed to a VM. For example, if there are two ports belong to a specific IOMMU group, they can only be passed together but which is not we want to do here, so we need to make sure the devices are in seprate groups. If not, it might be due to [ACS](https://www.reddit.com/r/VFIO/comments/5h9wyo/can_someone_explain_acs_to_me) not being supported well either by CPU or chipset.

<br>

## Create VFs
It's time to make expected VFs for OpenStack. Create the VFs via the PCI SYS interface:
 
{% highlight properties %}

$echo 4 > /sys/class/net/eth1/device/sriov_numvfs

{% endhighlight %}

To make it persistent even after boot, append the above command to `rc.local` file located in `/etc/rc.d/` directory. OS executes the `rc.local` script at the end of the boot process.

##Note:
When we try to change the amount of VFs, we might receive the error `Device or resource busy`. In this case, set **the value** to **0** before change it to the value. 

Run below commands to check VFs are successfully created:

{% highlight properties %}

$lspci | grep Ethernet

{% endhighlight %}

The output should display Virtual Function device.

One more thing to make sure is the interface shouldn't be **down**, otherwise the instance will fail to spawn:

{% highlight properties %}

#ip link set eth1 up

{% endhighlight %}

<br>
## Whitelist PCI devices(nova-compute) on compute node

Tell **nova-compute** service which pci devices are allowed to be passed through. Edit `nova.conf`:

{% highlight json %}

[default]
pci_passthrough_whitelist = { "devname": "eth1", "physical_network": "physnet1"}

{% endhighlight %}

with this parameter, all VFs belonging to **eth1** are allowed to be passed through to VMs and belong to the provider network **phsynet1**.

Alternately, two other parameters can be used for whitelist, and the information can be found by `lspci` command. For instance:

* PCI **address**

{% highlight json %}

pci_passthrough_whitelist = { "address": "*:0a:00.*", "physical_network": "physnet1"}

{% endhighlight %}
The **address** can be specified more detail to separate 

* PCI **vendor_id** and **product_id**<span style="font-size:0.7em;">(`8086` : Intel, `1572` : X710 card)</span>

{% highlight json %}
pci_passthrough_whitelist = { "vendor_id": "8086", "product_id": "1572", "physical_network": "physnet1"}
{% endhighlight %}

<br>

## Attach VFs to VM
There are two ways to attach VFs to an instance. You can create ports or use pci_alias

- Method 1: SR-IOV Ports  
Create an network and subnet for the **physnet1**. SR-IOV ports will be attached to this network. For example:

<pre>

$ openstack network create --provider-physical-network <b>physnet1</b> \
    --provider-network-type flat  \
    sriov-net 

$ openstack subnet create --network sriov-net \
    --subnet-pool a-subnetpool \
    sriov-subnet

</pre>

Create a port and launch an instance with it.

<pre>
$ openstack port create --network sriov-net  <b>--binding:vnic_type</b> <b>direct</b> sriov-port

$ openstack server create --flavor m1.tiny --image centos --wait --nic port-id=<b>$port_id</b> test-sriov
</pre>

<br>
- Method 2: PCI Alias  
To use PCI alias, some of OpenStack services should know it first. Add `pci_alias` line to nova.conf of each service listed below:
<pre>
<b>pci_alias = {"vendor_id":"8086", "product_id":"1572", "device_type":"type-VF", "name":"a1"}</b>

[Controller node]
* nova-api, nova-scheduler

[Compute node]
* nova-compute

</pre>

The difference to whitelist is only `vendor_id` and `product_id` can be used to specify which device to be aliased. After attach the line to the files, restart the services. 

<br>

Let's configure a flavor to request 4 VFs:

<pre>
$ openstack flavor set m1.tiny --property "<b>pci_passthrough:alias</b>"="<b>a1:4</b>"
</pre>

Launch the instance with the flavor.
```
$ openstack server create --flavor m1.tiny --image centos --wait test-sriov
```
