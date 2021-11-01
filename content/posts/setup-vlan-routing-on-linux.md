---
title: 'Setup VLAN routing on Linux'
date: 2021-10-26T15:30:38+03:00
draft: false
tags:
  - networking
  - Linux
  - iptables
keywords:
  - networking
  - Linux
  - iptables
---

# What is the purpose of this writing?

As for the 25 April 2021, I was in the process of learning networking for the DevOps Engineer position in my current company at Namecheap and I've received the task to understand networking.

The task was about creating a network configuration where one Linux machine is acting as a router and a few others are like clients that can communicate with each other(depends on the topology) or go to the Internet only via that router. I just want to answer the questions that appeared during development and understand the topic even more deeply.

You may have heard about Feynman's technique of teaching where you are trying to explain the topic as simply as possible. So, in general, the post is an attempt at learning in a different way. Also, a great way of memorization because I can get back to this article at any time.
Covered topics

- virtualization
- `iptables` and firewalls
- Virtual box
- virtual machine
- NAT
- OSI layers
- network interface

## Actual task

What I've received?
And then I started digging into the topic.
Questions?

1. What is VLAN?
2. How to make the connectivity?
3. How can VLANs communicate?
4. How to set everything up in the tools like Virtual Box?
5. and tons of others.

Let's start with configuring network interfaces on the router.

## What shall I do on the router machine?

So, what is a network interface? Physically, the network interface is a place where you plug an ethernet cable into the computer(similar to WiFi).
This network card could be emulated by the virtualization technologies and on one real network interface, you can create different virtual network interfaces.

> Virtualization technologies could give different capabilities for network interfaces inside their virtualmachineland.

So we understand that there must be 2 network interfaces for the router and one for each of the VMs.

### Choosing network interfaces

In order to accomplish this task, I need to set up 5 virtual machines in the virtualized environment and I choose the Virtual Box.

Virtual box and Virtualization technologies provide hardware resources by writing programs and software. Your machine will work completely in the same way, as it was working in the real environment without a virtualization layer. Except, the hypervisor will consume some resources to run your VMs.

So, this BOX is taking care of some basic things needed for the operating system inside this box to work. It creates a virtual hard drive, sets up network interfaces, allows you to boot CD into the virtual CD-ROM.

Knowing that I assume that the router should have two virtual network interfaces(one for communicating with the internet, second for processing all routes related to the clients) and other machines should have only one. Now it is time to decide what Virtual box provides us and what suites to our purpose.

The long story is placed [here](https://www.virtualbox.org/manual/ch06.html). Long story short: I needed host-only networking to create a network of virtual servers hidden behind this box.
Using that Box analogy, let VMs play inside without any connections to the outside world. You will need to create a network in the Virtual box settings and attach the interfaces of your machines to it.
Very detailed explanation [is available via this link](https://www.nakivo.com/blog/virtualbox-network-setting-guide/)

Then goes the next step of choosing the second interface to the router. I understood from the task that I need something with NAT but NAT network or just NAT? Carefully look at the documentation. After a few minutes of staring, I saw that just NAT does not allow vm-to-vm communication.
Now we are finished with network interfaces.

> It is not possible to manage networking settings inside a VM, it is made outside by the virtualization technology. Similarly, you define settings in VMWare.

You can combine the properties of two net interfaces on one machine to get value from both. But combination always increases complexity. Make sure that you deeply understand the building blocks. They will come in handy in different situations like working in the Cloud, creating your own network at work or at home.

## Vlans

Once we set up our virtual machines, we can move further with all routing and VLANs.
We need to understand what is VLAN.

I imagine VLAN as a virtual(as the name stands) building that stands on this "real" ethernet. And depends on the structure of this building, packets to go one place or to another.

Note that VLANs exist only on the router, a client just contacts the router via default routing and that it. VLAN id is needed when two routers communicate on the VLAN level that is level 2 of the OSI model.
Speaking more technically, VLAN is a logical entity that provides segmentation of the network usually done in the corporate environment. Such segmentation allows restricting users in one department to contact very sensitive resources. In our task, we want to accomplish exactly such a purpose.

Also, need to admit that since it provides logical segmentation, it does not implement any restrictions. Restrictions are about firewalling. Please see the image of the router once again, it has interfaces in all VLANs and can route traffic from and to them.
It means that they must be used in conjunction
Firstly, you need to create a traffic flow via the router.
What is needed for routing? IP addresses of course!

So computers can communicate with each other on level 3 of OSI and you can ping each machine.
I will give the following CIDRs to the machine and particular IP for interface and machine.

- vlan1 10.1.1.0/24
- vlan2 10.1.2.0/24
- vlan3 10.1.3.0/24
- vlan0 10.1.4.0/24

Why not only the one IPs? Because I will be able to add a new machine to the VLAN with the IP from range easily. Management of single ip and of the CIDR is completely the same.

![image of configuration with CIDRs assigned](/vlan_images/Networking_practice.jpg)


### What is CIDR?

CIDR block is the combination of the network part of ip address and how many ips are left available for assignment to the host machine in the subnet. 24 means that the first 24 digits from IP is filled and, 8 is left available. It equals 254 IPs available.

### Router?

A router is a machine that is present in all of the subnets, its main task to forwarding packets to the subnet.
The presence of a router in the subnet is indicated by presenting the network interface with the ip.

To configure all this stuff, you will need only one command `ip`. Make sure to read the man page carefully. Especially for `ip route`, `ip address`, `ip link`.

### Make the configuration persestent

- Ubuntu -> network-manager
- Centos -> network interface configurations.

These config files are read at a startup and are persistent from reboot to reboot.

You can use the virtualization option to save the machine state if you are not sure. Also, you should know about snapshots. You can make a snapshot per any change in the configuration. Similar to the commit in git, creating a point to get back to the working state.

## Firewalling

The next part is about network firewalls and some restrictions.
The best tool to configure firewall rules is `iptables`.

New tools like `ufw` for ubuntu or `firewall-cmd` on Centos are some kind of simplification and wrappers around `iptables`.
But with the simplification goes the limitations of a variety of cases. They are fine for something usual, but require the same `iptables` rules to be written in firewall-cmd direct rule(which didn't work at all in my case) or writing config files for `ufw` with the same iptables rules as you were writing directly.

In general, these new tools are great and easy to use but only for "usual" tasks and not for something complicated.

## How do iptables work?

Basically, it inspects all incoming packages that are processed and seen by the Linux kernel and can do some things based on the information in the package.

`iptables` has different algorithms of checking types of packages and different actions to perform like NATing the request, dropping packages or forwarding them.
There are 5 chains speaking in terms of iptables:

- forward
- input
- output
- prerouting
- postrouting

Once again, no configurations are made on the end virtual machine, everything is done on the router.

Also, you may see that routers usually have a special type of image in diagrams and so on. We can think of them in this way. And the kernel is the traffic light that tells if this package can go right or left :)
  
Now we need to write rules for each VLAN that has its own range of IPs.

## NAT

NAT stands for **Network address translation**.
It is the process of changing the IP address from one package to another. Why it may be useful? To hide the servers behind NAT. NAT gateway will be contacted by these servers behind the interface present in the subnet.

In our case, we want exactly that, to hide servers behind and allow only one end machine to contact the internet.

Knowing that, we will write a rule that will forward traffic to another interface on the router(do you remember when we were choosing them precisely?) that can go to the internet by default.

The package must go from one interface that received a package to another that will send the package. Even if it is done on one computer, it is a postrouing chain in iptables.

Please notice that packets go in both directions. Only one allow rule for outcoming requests will not make the work. You will need another rule for incoming packages.

Saving configuration to persistent and we are done.

## Conclusion

I've tried to explain all the stuff in the analogs manner because I think they stick to your mind and you can reuse them in the future.

If you, as I ended up, opened about 15 tabs with different explanations of terminologies, try to be calm and process them one by one understanding each small piece will make all of them a great puzzle.

What helped personally me with the number of unfamiliar things that I offloaded them to the "paper".(I've used iPad :) ) and see them from the side. Because our brain can't keep in "RAM" a lot of things at one time, drawing or writing them down helps you to process them. That's why whiteboard soft became popular in COVID time and due to interactive meetings too

This article is the process of my digging for about a month in network configuration and I could have made some mistakes in text or logic or anything else, please tell me about it. It is my experiment and first writing at all, I will be happy to read any feedback.

If this article saved you some time, you can ![test](/images/DV_buy_me_acoffee_picture.jpg) [buy me a coffee](https://www.buymeacoffee.com/worldcompass).

## P.S
Note that actual commands are not pasted. Maybe I will regret it later, but it is made intentiously.
And to tell the truth, they are not complicated at all. 
