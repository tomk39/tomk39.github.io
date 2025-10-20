---
layout: post
title: "Deploy a 5-Router Lab in 15 Minutes on Juniper SRX 100"
categories: ["Juniper"]
excerpt_separator: <!-- excerpt -->
---
<p align="center">

<img src="/images/juniper_virtlab/1.png" alt="Juniper-SRX" title="Juniper-SRX-top">  

</p>

In this post, we’ll show you exactly how to deploy a fully connected 5-router topology in less than 15 minutes. 
Transforming your Juniper SRX 100 from an old forgotten box into a high-speed engine for learning and validation.
<!-- excerpt -->
<p>

</p>  
Before we begin, here is what you’ll need to execute this fast deployment:
* Core Hardware: Your Juniper SRX 100 device.  
* Connectivity: Patch cables.  
* Time: A clear 15-minute window for the configuration process.  

Let's dive in!  

To deploy our 5-router lab in minutes, we will follow these three quick steps:  

**<a href="#section1">1. Loop two physical ports on SRX.</a>**    
**<a href="#section2">2. Configure Packet Mode and Disable Security Flow on Juniper SRX.</a>**  
**<a href="#section3">3. Create the 5-Router Topology with Virtual Router Instance type.</a>**  

The target lab design topology is shown in Figure 1. below.

<img src="/images/juniper_virtlab/2.png" alt="Juniper SRX" title="Juniper SRX Lab">  
Figure 1. Lab Design Topology.  



**<span style="color:#074080; font-size:18.0pt;" id="section1">1. Loop two physical ports on SRX.</span>**  

To create a 5-router lab on a single SRX, we need to create a closed loop that allows traffic to pass between our different virtual instances. Since the SRX 100 has limited interfaces, we'll use a two physical ports with vlan tag sub-interfaces.  

**The Physical Connection**


Use a standard patch cable to connect two adjacent physical ports on your SRX 100. For this lab, we'll use:

* fe-0/0/6
* fe-0/0/7  

<img src="/images/juniper_virtlab/3.png" alt="Juniper SRX" title="Custom UTP cable for loop">  
Figure 2: The custom UTP CAT 5 patch cord used for the physical loop.

```
set interfaces fe-0/0/6 flexible-vlan-tagging
set interfaces fe-0/0/6 fastether-options auto-negotiation  
set interfaces fe-0/0/7 flexible-vlan-tagging
set interfaces fe-0/0/7 fastether-options auto-negotiation 
commit
``` 

This physical connection will act as our high-speed, internal backbone fabric, allowing the virtual routers what we will create later to talk to each other without needing external equipment.  

<img src="/images/juniper_virtlab/4.png" alt="Juniper SRX" title="Juniper SRX Lab">  
Figure 3. SRX Ports Looped with UTP Cable.

**<span style="color:#074080; font-size:18.0pt;" id="section2">2. Configure Packet Mode and Disable Security Flow Juniper SRX.</span>**  
The SRX is a firewall, meaning it runs in flow mode by default, inspecting traffic and requiring security policies and zones. To turn it into a dedicated, multi-router platform that simply forwards packets, we must switch it to packet mode.

⚠️ Warning: This step removes all firewall functions and security services.  

**Switching the SRX to Router Mode (In Next 1 minute if we will not calculate SRX boot time)**
* Delete Existing Security Configuration: We first wipe all pre-existing security settings that would conflict with packet mode.

```
delete security
```
* Enable Packet Mode: We instruct the Packet Forwarding Engine (PFE) to process traffic on a per-packet basis, like a traditional router. 
I like to enable for all address families.  

```
set security forwarding-options family inet mode packet-based
set security forwarding-options family inet6 mode packet-based
set security forwarding-options family mpls mode packet-based
set security forwarding-options family iso mode packet-based
```
* Commit and Reboot, this change is fundamental and requires a system reboot to take effect.  

```
commit and-quit
request system reboot
```
(Go grab a coffee while it reboots! You've already completed the hardest part.)   

Now that our SRX is a pure router, we can segment it into five distinct logical routers using Virtual Router (VR) instances. A VR instance is a type of routing instance that creates completely separate routing tables and forwarding domains.  

**<span style="color:#074080; font-size:18.0pt;" id="section3">3. Create the 5-Router Topology with Virtual Router Instance type.</span>**   

**Topology Goal**
We will create five isolated routing domains (AS1 through AS4 and PE) and then connect them using logical interfaces to form a virtual network.

**The Virtual Router Configuration (Next 5 minutes)**

```
set routing-instances AS1 instance-type virtual-router
set routing-instances AS2 instance-type virtual-router
set routing-instances AS3 instance-type virtual-router
set routing-instances AS4 instance-type virtual-router
Global routing table on Juniper SRX will represent PE.
```

<img src="/images/juniper_virtlab/5.png" alt="Juniper SRX" title="Juniper SRX Virtual Routers">  
Figure 4: Representing logical connectivity between routing instances.

**Next Step: Connecting the Virtual Routers**

Below you can find the interface configuration for fe-0/0/6 and fe-0/0/7:  

**fe-0/0/6**
```
set interfaces fe-0/0/6 unit 12 vlan-id 12
set interfaces fe-0/0/6 unit 12 family inet address 10.90.12.2/30
set interfaces fe-0/0/6 unit 13 vlan-id 13
set interfaces fe-0/0/6 unit 13 family inet address 10.90.13.2/30
set interfaces fe-0/0/6 unit 23 vlan-id 23
set interfaces fe-0/0/6 unit 23 family inet address 10.90.23.2/30
set interfaces fe-0/0/6 unit 24 vlan-id 24
set interfaces fe-0/0/6 unit 24 family inet address 10.90.24.2/30
set interfaces fe-0/0/6 unit 113 vlan-id 113
set interfaces fe-0/0/6 unit 113 family inet address 10.90.113.2/30
set interfaces fe-0/0/6 unit 114 vlan-id 114
set interfaces fe-0/0/6 unit 114 family inet address 10.90.114.2/30
```  
**fe-0/0/7**
```
set interfaces fe-0/0/7 unit 12 vlan-id 12
set interfaces fe-0/0/7 unit 12 family inet address 10.90.12.1/30
set interfaces fe-0/0/7 unit 13 vlan-id 13
set interfaces fe-0/0/7 unit 13 family inet address 10.90.13.1/30
set interfaces fe-0/0/7 unit 23 vlan-id 23
set interfaces fe-0/0/7 unit 23 family inet address 10.90.23.1/30
set interfaces fe-0/0/7 unit 24 vlan-id 24
set interfaces fe-0/0/7 unit 24 family inet address 10.90.24.1/30
set interfaces fe-0/0/7 unit 113 vlan-id 113
set interfaces fe-0/0/7 unit 113 family inet address 10.90.113.1/30
set interfaces fe-0/0/7 unit 114 vlan-id 114
set interfaces fe-0/0/7 unit 114 family inet address 10.90.114.1/30
```

**Assigning interfaces to virtual router instance AS1**
```
set routing-instances AS1 interface fe-0/0/7.12
set routing-instances AS1 interface fe-0/0/7.13
set routing-instances AS1 interface lo0.1
```  

**Assigning interfaces to virtual router instance AS2**
```
set routing-instances AS2 interface fe-0/0/6.12
set routing-instances AS2 interface fe-0/0/6.23
set routing-instances AS2 interface fe-0/0/7.24
set routing-instances AS2 interface lo0.2
```  

**Assigning interfaces to virtual router instance AS3**
```
set routing-instances AS3 interface fe-0/0/6.13
set routing-instances AS3 interface fe-0/0/7.23
set routing-instances AS3 interface fe-0/0/7.113
set routing-instances AS3 interface lo0.3
```  

**Assigning interfaces to virtual router instance AS4**
```
set routing-instances AS4 interface fe-0/0/6.24
set routing-instances AS4 interface fe-0/0/7.114
set routing-instances AS4 interface lo0.4
```  
The final result of the configuration is shown in Figure 5.  

For routing and inter-instance communication, your newly created virtual routers support a wide range of protocols, including:  

* Static Routing
* OSPF (Open Shortest Path First)
* ISIS (Intermediate System to Intermediate System)
* BGP (Border Gateway Protocol)
* MPLS (Multi-Protocol Label Switching)
* With support for IPv4 and IPv6.

<img src="/images/juniper_virtlab/6.png" alt="Juniper SRX" title="Juniper SRX Virtual Routers">  
Figure 5. 5 Routers network topology.  

**The 15-Minute Network is Live!**

Congratulations! You've successfully converted your single SRX 100 into a complex 5-node routing lab, all within the time it takes to brew a cup of tea. This setup proves that complex labs don't require complex hardware. Use this topology to master OSPF, BGP, or MPLS.

**Go build something great!**

<a href="#top">Back to Top</a>   



