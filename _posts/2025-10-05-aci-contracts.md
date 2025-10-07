---
layout: post
title: "Understanding Contracts in Cisco ACI: The Backbone of Network Security"
categories: ["Cisco ACI"]
excerpt_separator: <!-- excerpt -->
---
<p align="center">
<img src="/images/aci_contracts/1.png" alt="ACI-top" title="ACI-top"> 
</p>

We recently explored the foundational elements of Cisco ACI, diving into how Tenants, VRFs, 
Bridge Domains, and EPGs serve as the logical building blocks for organizing your network. 
We saw how these components create a structured environment for applications, but that raises a critical question, how do these EPGs actually talk to each other securely?
<!-- excerpt -->
<p>
How do you define precisely what traffic is allowed, and what is denied, between different application tiers or external networks? This is where Cisco ACI Contracts come in. They are the core enforcement mechanism for defining communication policies and building a 
zero-trust security model within your ACI fabric.
</p>  

In this post, we'll cover the different types of contracts and policy enforcement options: 

**<a href="#section1">1. Standard Contracts: Building the ACI Security Whitelist</a>**    
**<a href="#section2">2. Taboo Contracts: ACI's Blacklist for Overriding Security Policies.</a>**  
**<a href="#section3">3. vzAny Contracts: The Shortcut to allow communication inside VRF.</a>**  
**<a href="#section4">4. Preferred Groups: When to Unenforce Policy in ACI.</a>**  
**<a href="#section5">5. Unenforced VRFs: The Full Open Communication inside VRF.</a>**  


**<span style="color:#074080; font-size:18.0pt;" id="section1">1. Standard Contracts: Building the ACI Security Whitelist.</span>**  
<div style="text-align: justify;">
The Contract is the core policy object in Cisco ACI that enables inter-EPG communication based on a provider-consumer model. It defines the exact, permitted traffic flows, serving as the explicit allow-list that overrides the fabric's default deny-all. A standard contract is composed of several key components:  
</div>  


1. **Filters** are the most granular rules, specifying permitted traffic based on network attributes such as EtherType, IP, ICMPv4 and v6 Source and Destination Port/Range.

2. **Subjects** are Logical containers for Filters that define the directionality of the traffic flow. It also defines the directionality of the communication using options like "Apply Both Directions" and "Reverse Filter Ports" these “Subject” options are responsible for automatically creating the necessary return path TCAM entry (ACL rule) for a connection.  

3. **Scopes** Settings (Application Profile, VRF, Tenant, and Global) that establish the contract's domain, controlling how widely it can be shared and reused across the ACI environment.  
    * **Application Profile scope -** communication policies that are strictly internal to a single application, ensuring that the contract is not accidentally used by other applications, even in the same VRF.  
   * **VRF scope -** The default and most common scope for defining communication policies between EPGs that share a Layer 3 forwarding domain. This scope is used for most intra-VRF contracts.  
   * **Tenant Scope -** Primarily for Inter-VRF (route leaking) communication when both VRFs are in the same tenant. It allows EPGs in one VRF to consume a contract provided by an EPG in a different VRF, as long as they are in the same tenant.  
   * **Global scope -** Inter-Tenant contracts, where the provider and consumer are in different tenants. For contracts defined in the Common Tenant that need to be universally available (e.g., contracts for shared services like DNS/NTP). or Inter-VRF communication where the VRFs are in different tenants.  

4. **Action** specifies whether to permit, deny, or log the traffic (deny actions are supported in APIC Release 3.2(x) and later with EX/FX switches).  

As many are still struggling with ACI contract direction (Consumer to Provider and Provider to Consumer), let me reveal one secret:
* **Consumer is the SOURCE** 
* **Provider is the DESTINATION**  

Apply your policies using this simple traffic flow rule, and you’ll get it right every time!  

Below figures 1 through 3 illustrate the operational behavior of the "Apply Both Directions" and "Reverse Filter Ports" settings.

<img src="/images/aci_contracts/1.gif" alt="ACI-Contracts1" title="ACI-Contracts1"> 

Figure 1. Applying contract between two EPGs.

The Consumer EPG can only initiate a new session with the Provider EPG on the specified port, which in this case is TCP 22 for SSH.  
However, communication won't be established because the contract is currently applied only in the **Consumer-to-Provider** direction, causing the necessary return traffic to be dropped. Therefore, we have two options: 
* either create a second contract in the **Provider-to-Consumer** direction or 
* use the "Apply Both Directions" option with "Reverse Filter Ports."  

Since contracts utilize hardware resources on the leaf switches, I recommend using "Apply Both Directions" with "Reverse Filter Ports."

<img src="/images/aci_contracts/2.gif" alt="ACI-Contracts1" title="ACI-Contracts1"> 

Figure 2. Feature "Apply Both Direction" Enabled.

With "Apply Both Directions" (Option ON):
* The Consumer EPG can initiate a connection to the Provider EPG.
* The Provider EPG can also initiate a new, separate connection back to the Consumer EPG on the same port(s).

Based on Figure 2. SRV1 and SRV2 can open SSH session to DB1 and DB2 and vice versa DB1 and DB2 can open SSH session to SRV1 and SRV2 but communication won't be established.

<img src="/images/aci_contracts/3.gif" alt="ACI-Contracts1" title="ACI-Contracts1"> 

Figure 3. Representing establishment of SSH session from Jumphost (SRV1 and SRV2) to DB servers.

**Note:**  
Contracts can be applied only on unicast traffic. Broadcast, Unknown unicast and Multicast Protocols (BUM) traffic and below listed protocols are implicitly permitted.  
* ARP reply (unicast)
* DHCP v4 (prot 0x11, sport 0x44, dport 0x43)
* DHCP v4 (prot 0x11, sport 0x43, dport 0x44)
* DHCP v6 (prot 0x11, sport 0x222, dport 0x223)
* OSPF (prot 0x59)
* EIGRP (prot 0x58)
* PIM (prot 0x67)
* IGMP (prot 0x2)
* ND-Sol ICMPv6 (prot 0x3a dport 0x0087)
* ND-Advt ICMPv6 (prot 0x3a dport 0x0088)

Moreover, in case of Multi-Site deployment, Cisco Multi-Site Orchestrator (MSO) creates one contract for each subject.

<div style="text-align: justify;">
</div>
<a href="#top">Back to Top</a>   

**<span style="color:#074080; font-size:18.0pt;" id="section2">2. Taboo Contracts: ACI's Blacklist for Overriding Security Policies.</span>**  

Taboo contracts are specialized contracts used to explicitly deny specific traffic for a single EPG. Unlike standard contracts, they are:
* **Unidirectional -** Always provided by the EPG, with no consumer relationship.
* **Higher Priority -** Evaluated before standard contracts in the hardware’s Policy CAM.
* **Limited Scope -** Applied to a single EPG to block specific traffic, such as insecure protocols.
* **Taboo contracts** are not supported with Endpoint Security Groups (ESGs).
* **They cannot be applied to vzAny.**
* **Cisco best practices** generally advise against its regular use in production environments and suggest to use standard contracts with deny action if it is needed.

**<ins>Example Use Case:</ins>**  

1. You want to block HTTP (port 80) traffic to Web_EPG while allowing other traffic via standard contracts.
2. Create a taboo contract Deny_HTTP_Taboo with a filter for TCP port 80 and apply it as provided by Web_EPG.

<a href="/images/aci_contracts/5.png" target="new" title="click here to see the full sized image"><img src="/images/aci_contracts/5.png" alt="ACI-Taboo"></a>  

Figure 4. A Taboo-Contract blocking access to HTTP (insecure TCP port 80).

<a href="#top">Back to Top</a> 


**<span style="color:#074080; font-size:18.0pt;" id="section3">3. vzAny Contracts: The Shortcut to allow communication inside VRF and more.</span>**  

vzAny is a powerful construct that simplifies the application of security policies across all Endpoint Groups (EPGs) and Endpoint Security Groups (ESGs) within a Virtual Routing and Forwarding (VRF) instance.  

**Key characteristics of vzAny:**  

**1. VRF-Specific -** vzAny is scoped to a single VRF and cannot directly apply to EPGs/ESGs in other VRFs.  
**2. Contract Simplification -** Instead of applying a contract to each EPG/ESG individually, vzAny applies it to all EPGs/ESGs in the VRF simultaneously. **Remember vzAny representing all EPGs inside VRF.**  
**3. Provider or Consumer Role -** vzAny can act as a contract provider (offering services) or consumer (accessing services), depending on the configuration.  
**4. Standard Contracts Only -** vzAny supports standard contracts but not taboo contracts, which are limited to individual EPGs.  

<img src="/images/aci_contracts/2.png" alt="ACI-vzAny2" title="ACI-vzAny2">  

Figure 5. Representation of applying the contract per EPG.  


<img src="/images/aci_contracts/3.png" alt="ACI-vzAny3" title="ACI-vzAny3">  


Figure 6. Representing of applying the contract to vzAny.  

Under the "Application Profile" and "Topology" tabs, you can view the graphical representation of the relationship between EPGs and Contracts.
Below Figure 6 illustrates how DNS and NTP services are shared for all EPGs within the VRF using vzAny.  

**Note** the checkbox on the right, "Show VRF in EPG," which helps us identify which VRF a particular EPG belongs to.  

<a href="/images/aci_contracts/4.png" target="new" title="click here to see the full sized image"><img src="/images/aci_contracts/4.png" alt="ACI-vzAny4"></a>  

Figure 7. Contract relationship: vzAny consumes services provided by DNS and NTP EPGs. 



**Please Note:**
External EPGs that are associated with L3Outs and are part of a VRF are also included in the vzAny logical group.

**Limitations of vzAny:**  
**1. VRF Scope -** vzAny is limited to a single VRF. It cannot directly apply contracts to EPGs/ESGs in other VRFs without additional configuration (e.g., contract export or shared services).  
**2. No Taboo Contracts -** vzAny does not support taboo contracts, which are used to deny specific traffic for individual EPGs.  
**3. Over-Permissiveness Risk -** Applying contracts to vzAny affects all EPGs/ESGs in the VRF, which may lead to unintended traffic flows if not carefully planned.  
**4. Not for Granular Control -** For specific EPG-to-EPG policies, individual contracts are preferred over vzAny.  


<a href="#top">Back to Top</a>  


**<span style="color:#074080; font-size:18.0pt;" id="section4">4. Preferred Groups: When to Unenforce Policy in ACI.</span>**  

A Preferred Group is a VRF-level configuration that designates a subset of EPGs or ESGs within a VRF as members of a group that can communicate with each other without contracts. When an EPG or ESG is added to the Preferred Group, it can send and receive traffic to/from other Preferred Group members within the same VRF, bypassing the need for standard contracts, provided the VRF is in enforced mode. I would like to mentioned that only one group can exist per VRF.

**Limitations of Preferred Groups:**
1. Preferred Groups are limited to a single VRF. 
2. No Taboo Contract Support: Taboo contracts, which explicitly deny traffic for individual EPGs, cannot be overridden by Preferred Groups. If a taboo contract is applied to an EPG, it takes precedence.
3. All-or-Nothing Communication, preferred Group members can communicate freely with each other for all traffic types.
4. Not for External Connectivity: Preferred Groups do not apply to L3Out external EPGs for external connectivity; contracts are still required.
5. Policy CAM Usage: Preferred Group rules consume Policy CAM on leaf switches, which can impact scalability in large deployments.
6. If the VRF is unenforced, contracts are bypassed, and the Preferred Group has no effect (all EPGs/ESGs can communicate freely).

<a href="#top">Back to Top</a>  


**<span style="color:#074080; font-size:18.0pt;" id="section5">5. Unenforced VRFs: The Full Open Communication inside VRF.</span>**  

An unenforced VRF is a VRF configuration in Cisco ACI where the policy enforcement is disabled.


**In this mode:**  
1. All EPGs and ESGs within the VRF can communicate with each other without requiring contracts.  
2. The unenforced setting applies only to intra-VRF traffic; inter-VRF communication still requires contracts and route leaking.  
3. Traffic flows freely between endpoints within the VRF.  
4. Contracts, including standard and taboo contracts, are bypassed for intra-VRF communication.  
5. Features like Preferred Groups and vzAny become irrelevant in an unenforced VRF, as all traffic is permitted.  

| Feature | Preferred group | vzAny | Unenforced VRF |
| :--- |:--- | :--- |
| Scope | Specific EPGs/ESGs in a VRF | All EPGs/ESGs in a VRF | All EPGs/ESGs in a VRF |
| Contract Requirement | No contracts needed between members. | Contracts required (provider/consumer) | No contracts needed for any traffic| 
| Granularity| Selective (only members communicate freely). | Applies to all EPGs/ESGs via contracts. | No control (all traffic allowed) |
| Inter-VRF Support | No (requires contracts for inter-VRF). | Supports inter-VRF with contract export. | No (VRF isolation still applies) |
| Use Case | Controlled open communication within VRF. | Simplified contract application. | Fully open VRF (less secure) |

Table 1. Representing comparison Preferred Groups vs. vzAny vs. Unenforced VRF.

<a href="#top">Back to Top</a>  

