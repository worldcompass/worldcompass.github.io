---
title: 'Setup VLAN routing on Linux'
date: 2021-10-26T15:30:38+03:00
---

## What is the purpose of this writings?
As for the 25 April 2021, I was in the process of learning networking for the DevOps position in my current company at Namecheap and I've received the task to understand networking.

The task was about creating network configuration where one linux machine is acting like a router and a few others are like clients that can communicate with each other(depends of the topology) or go to the Internet. I just want to answer the questions that rised during development and understand the topic even more deeply.

You may have heard about Feynman technique of teaching where you are trying to explain the topic as simply as possible. So, in general, there topics are the attempt for learning in a different way. Also, a great way of memorization because I can get back to this article at any time.

Covered topics
- virtualization
- iptables and firewalls
- virtual box
- virtual machine
- NAT
- OSI layers
- network interface

## Actual task
What I've received?
![[Networking practice - Frame 1.jpg]]

And then I started digging into the topic.
Questions?
1. What is VLAN?
2. How to make the connectivity?
3. How can VLANs communicate?
4. How to set everything up in the tools like Virtual Box?
5. and tons of others.

Let's start with configuring network interfaces on the router.

## What shall I do on the router machine?
So, what is [[network interface]]? Network interface is a place where you plug ethernet cable into computer(similar for WiFi).

This network card could be emulated by the virtualization technologies and on one real network interface, you can create different virtual network interfaces.

> Virtualization technologies could give different capabilities for network interfaces inside their virtualmachineland.
> Also, there could be different types of them.

So, from the task, we understand that there must be 2 network interfaces for the router and one for each of the vms.

---
### Choosing network interfaces
In order to accomplish this task, I need to set up 5 [[virtual machine]] in the virtualized environment and I choose the [[Virtual Box]]. 

image with box goes here.

Virtual box and [[Virtualization]] technologies provides hardware resources by writing programs and software. Your machine will work completely in the same way, as it was working on the real environment without virtualization layer.

So, this BOX is taking care of some basic things needed for the operation system inside this box to work. It creates virtual hard drive, sets up network interfaces, allows you to boot CD into the virtual CD-ROM.

Knowing that, I assume that router should have two virtual network interfaces and other machines should have only one. Now it is time to decide what Virtual box provides us and what suites to our purpose. 
Long story is placed [here](https://www.virtualbox.org/manual/ch06.html). Long story short: I needed host-only networking to create a network of virtual servers hidden behind this box.
Using that Box analogy, let VMs to play inside without any connections to the outside world. You will need to create a network in the Virtual box settings and attach interfaces of your machines to it.
Very detailed explanation [here](https://www.nakivo.com/blog/virtualbox-network-setting-guide/)

Then goes the next step of choosing the second interface to the router. I understood from the task that I need something with [[NAT]] but NAT network or just NAT? Carefully look at the documentation. After a few minutes of staring, I saw that just NAT does not allow vm-to-vm communication.

Now we are finished with network interfaces.
- [ ] can I change network interface settings inside virtual box? Because I don't understand how it is made.

You can combine properties of two net interfaces on one machine to get value from both. But combination always increases complexity. Make sure that you deeply understand the building blocks. They will come handy in different situations like working in the Cloud, creating your own network at work or at home.

### Questions?
- What is virtualization in simple words?
- Why virtualizaiton soft emulate hardware in software?
- Why there are different network interface modes in virtual box?

---
## Vlans
Once we set up our virtual machines, we can move further with all routing and vlans.

We need to understand what is VLAN.

Image of ethernet frame with vlan id. another image how i imagine vlan on one interface.

I imagine vlan as virtual(as the name stands) building that stands on this "real" ethernet. And depends on the structure of this building, packets to go one place or to another.
Note that vlans exists only on router, client just contact the router via default routing and that it. Vlan id are needed when two routers communicate on the VLANs level that is level 2. 
Speaking more technically, vlan is logical entity that provide segmentation of the network usually done in the corporate environment. Such segmentation allows to restrict users from one department to contact very sensitive resources. In our task, we want to accomplish exactly such purpose.
Also, need to admit that since it provides logical segmentation, it does not implement any restrictions. Restrictions are about firewalling. Please see the image of the router once again, it has interfaces in all vlans and can route traffic from and to them. 

Firstly, you need to create a traffic flow via router.

image of when each can communicate with each other.

What is needed for routing? IP address, so computers can communicate with each other on level 3 of [[OSI layers|OSI]] and you can ping each machine.
I will give the following cidrs to the machine and particular ip for interface and machine.
- vlan1 10.1.1.0/24
- vlan2 10.1.2.0/24
- vlan3 10.1.3.0/24
- vlan0 10.1.4.0/24

Why not only the one IPs? Because I will be able to add a new machine to the vlan with the ip from range easily. Management of single ip and of the cidr is completely the same.

What is CIDR? cidr is the combination of network part of ip address and how many ips are left available assigned to the host machine. 24 means that first 24 digits from IP is filled and, 8 if left.

Image of infra with IPs assigned.

Router? Router is the machine that is located in all of the subnets, it's main task to forward packets to the subnet. 
This allocation of router in the subnet is indicated by presenting of the network inteface with the ip.

iamge of the routher here.

To configure all this staff, you will need only one command `ip`. Make sure to read the man page carefully. Especially for route, address, link.

> note for me: forgot how to make the configuration persistent.
### Make the configuration persestent
Ubuntu network-manager
Centos network interface configurations.

These config files are read at a startup and are persistent from reboot to reboot.

You can use the virtualization option to save the machine state if you are not sure. Also, you should know about snapshots. You can make as per any change in the configuration. Sort of git commit, creating a point to get back to the working state.

Maybe to write commands for each vm?

image of final configuration.

---
## Firewalling
Next part is about network firewall and some restrictions.

Best tool to configure firewall rules is `iptables`. New tools like `ufw` for ubuntu or `firewall-cmd` on Centos are some kind of simplification of the iptables. They are working above this program, but with the simplification goes the limitations of covered variety of cases. They are fine for something usual, but require same iptables rules to be written in firewall-cmd direct rule(which didn't work at all) or writing config files for ufw with the same iptables rules as you were writing directly. 

In general, these new tools are great and easy to use but only for a "usual" tasks and not for something complicated.

How does [[iptables]] work? Basically, it inspects all incomming packages that are processed and seen by the [[Linux kernel]] and can do some things based on the information in the package. It has different algorithms of checking types of packages and different actions to perform like NATing the request, dropping packages or forwarding them.
There are 5 of chains speaking in terms of iptables: forward, input, output, prerouting, postrouting. But, you can ommit that pre/post things, because at these steps inital packages are modified, that is not very often made. You can read more about them on any other resources. I would recommend
- [ ] place link here
- [ ] draw an image of it

Once again, no configuration are made on the end virtual machine, everything is done on the router. 

image of router as the cross road.

Also, you may see that routers usually have special type of image in diagrams and so on. We can think of them in this way. And the kernel is the traffic light that tells if this package can go right or left :)

Now we need to write rules for each vlan that has it's own range of IPs.

- [ ] will need to write rules for each vlan here

### NAT
Network address translation. It is the process of changing IP address in the package to another one. Why it may be useful? To hide the servers behind nat that will only have the private ips and nat gateway will be contacted by these servers behind.

In our case, we want exactly that, to hide servers behind and allow only one end machine to contact internet.

Knowing that, we will write a rule that will forward traffic to another interface on the router(do you remember when we were choosing them precisely?) that can go to the internet by default.

The package must go from one interface that received package to another that will send package. Even if it is done on one computer, it is postrouing chain in iptables.

Please notice that packets go in both direction. Only one allow rule for outcomming requests will not made the work. You will need another rule for incomming packages.

- [ ] nat rules go here

Saving configuration to persistent and we are done.


---

These article is the process of my digging for about a month in network configuration and I can made some mistakes in text or logic or anything else, please tell me about it. It is my experiment and first writing at all, I will be happy to read any feedback.
If this article saved your some time, you can (link to buy me a coffee).

I've tried to explain all the stuff in the analogues manner because I think they stick to your mind and you can reuse them in the future, or these images can evolve in your own pictures. 
If you, as I ended up, opened about 15 tabs with different explanations of terminologies, try to be calm and process them one by one understanding each small peace will make all them a great puzzle. 

When helped personally me with the amount of unfamiliar things that I offloaded them to the "paper".(I've used ipad :) ) and see them from the side. Because our brain can't keep in RAM a lot of things at one time, drawing or writing them down help you to process them. That's why whiteboard soft became popular in covid time. 

I've attached the file with my drawing below, they are mostly in russian and I wan't aware that I would like to share them publicly. If they could be useful and interesting for someone, I will try to make them in English.


## Summary
I am sure that I will split this big article in 3 parts, or even on 4 with the conclusion.

- [ ] try to add questions after each part for the reader to actively recall topics.
