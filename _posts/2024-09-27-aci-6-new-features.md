---
layout: post
title: Unlocking the Power of Cisco ACI 6.0
categories: ["Cisco ACI"]
---

<p align="center">
<img src="/images/aci_6_new_features/1.png" alt="ACI_60" title="ACI_60">   
</p> 
<p style='text-align: justify;'>
The Cisco ACI 6.0 release has been available for a couple of months now. Let’s check out what it brings to network engineers and IT professionals. Experience a new era of enhanced performance, security, and automation capabilities. In this blog, we’ll dive into the key enhancements of Cisco ACI 6.0 and explore how they can empower your network infrastructure.
</p>

**<span style="color:#074080">Main three features and enhancements in Cisco ACI 6.0.2 and 6.0.3</span>**   

1. Virtual APIC on VMware ESXi and Cloud 
2. VMM Integration with Nutanix AHV 
3. Nexus Dashboard - Connectivity analysis

**<span style="color:#074080">1. Virtual APIC  6.0(2)F</span>**   

- No physical APICs.
- No configuration limitations you can administer it like a regular APICs.
- Scale is pretty much same as for physical APIC.
- Limitations not able to run applications such as ELAM, policy explorer.
- APICs can be connected to leaf switches directly.
- Or they can be connected remotely via L3 network.
  During the initial setup, the wizard will ask you to choose how the APIC is connected to the network. Figure 1 below is an example.
- Virtual APIC Cluster Fully functional APIC that can manage up to 200 switches.
- 6.0(2)F Cloud hosted APIC Cluster can only be deployed within AWS Cloud.   
<p align="center">
<img src="/images/aci_6_new_features/2.png" alt="ACI 60" title="ACI 60"> 
</p>
**Figure 1** ACI setup wizard

**<span style="color:#074080">2. Integration with Nutanix AHV hypervisor  6.0(3)F</span>**
- APICs connect and authenticate to Prism Manager.
- Integration of Nutanix Acropolis Hypervisor (AHV) with Cisco Application Centric Infrastructure (ACI).
- This integration provides virtual and physical network automation and VM endpoints visibility in Cisco ACI.
- End to end network automation and end to end visibility.

**<span style="color:#074080">3. Nexus Dashboard - Connectivity Analysis 6.0(2)F</span>**

- Troubleshooting between two End points, src and dst IP addresses or src and dst MAC addresses.
- Visibility of what happening between control and data plane.
- Verification of control and data plane.
- Traces all possible forwarding paths between source and destinations.
- Identifies if there are problems in the path and what is causing issues.
- Helps to highlight real root cause, perform forwarding checks.

**<span style="color:#074080">4.Interface policy - simplify configuration</span>**
- This type of configuration is optional there is no need to change if you already master polices configuration. On that way you will keep everything under your control.
- This is more for new guys which is starting to use ACI Fabric.
- Guys with experience will not have so many benefits from this feature.
- It is compatible with previous manual configuration of polices.
- Not recommended for brown field deployments, I would like to recommend only for green field deployment.

**<span style="color:#074080">5.Weight-Based Symmetric PBR</span>**

In 6.0 it was introduced Weight-based PBR it means that we can send more traffic to specific node.
Use case:
•	On specific node we have connected Load Balancer.
•	On specific node we have more powerful firewall in the group.
•	Introducing new firewall, we can redirect small amount of traffic to new firewall for test purposes.

**<span style="color:#074080">6.Stretched L3out SVI across remote leaves.</span>**

- Supported for stretching a non-vPC L3Out SVI across remote leaf switches
Use cases:
- Mobile packet core.
- Containerized workloads special at the Edge.
- Cloud native network functions CNF.

**<span style="color:#074080">7. Multi-Site support for vzAny PBR and L3Out-to-L3Out PBR use cases.</span>**
From ACI 6.0(4)F and NDO 4.2(3) following uses cases are supported:
- vzAny-to-vzAny (Intra VRF).
- vzAny-to-EPG (Intra VRF).
- vzAny-to-L3out (Intra VRF).
- L3out-to-L3out ((Intra VRF) and inter-VRF).
- Standard EPG-to-EPG subnet under consumer still is required.
If you have plan to implement vzAny inside multi-site design, please check all details on link below:
<a href="https://www.cisco.com/c/en/us/td/docs/dcn/ndo/4x/configuration/cisco-nexus-dashboard-orchestrator-configuration-guide-aci-421/ndo-configuration-aci-use-case-vzany-pbr-42x.html"  target="_blank">Cisco Nexus Dashboard Orchestrator Configuration Guide for ACI Fabrics</a>.

**<span style="color:#074080">8. Main Routing improvements.ulti-Site support for vzAny PBR and L3Out-to-L3Out PBR use cases.</span>**

- BGP AS - Remove private AS from AS_PATH.
- Support for BFD on Secondary subnet IPv4/IPv6.
- BGP Additional Paths - Receive multiple paths for the same prefix without the new paths replacing any previous paths.

**<span style="color:#074080">9. Operating and Administering improvements.</span>**

ACI ver.6.0(1)F
- Simplifying interface configuration. 
- Secure Erase (RMA).

ACI ver.6.0(2)F
- Auto firmware update for Cisco APIC on discovery - APIC will be automatically upgrade on version same as fabric.
- Installing switch SMU without reload.
- Troubleshooting of QoS Polices.

ACI ver.6.0(3)F
- Reduce of time of APIC upgrade.
- Transaction-based Logging.
- Memory-based Automatic Switch Image Selection 32/64 bit.

ACI ver.6.0(4)F
- Streamlined navigation and editing of the stat collection threshold from a TCA fault.

**<span style="color:#074080">10. Main Security improvements</span>**

- Cisco Nexus 9K switch secure erase.
- Support for a user group map rule for SAML and OAuth2.
- Support for TLS ver 1.3 
- First hop security FHS support for VMM.
- TACACS external logging for switches.
- RSA Key 307 and 4096 support on X.509 certificates.

**<span style="color:#074080">11. Integrations.</span>**

- Enhanced LLDP TLVs for Azure Stack HCI.
- Policy mode for Cisco ACI and VMware NSX-T integration.
- VMM Domain support for VMware vSphere 8.0.


**<span style="color:#074080">12. Scale improvements.</span>**

<p align="center">
<img src="/images/aci_6_new_features/3.png" alt="ACI 60" title="ACI 60"> 
</p>

<p align="center">
<img src="/images/aci_6_new_features/4.png" alt="ACI 60" title="ACI 60"> 
</p>

<p align="center">
<img src="/images/aci_6_new_features/5.png" alt="ACI 60" title="ACI 60"> 
</p>


References:   
<a href="https://www.cisco.com/c/en/us/td/docs/dcn/aci/apic/6x/release-notes/cisco-apic-release-notes-611.html"  target="_blank">Cisco Application Policy Infrastructure Controller</a>   
<a href="https://www.cisco.com/c/en/us/td/docs/dcn/aci/apic/all/cisco-aci-releases-changes-in-behavior.html"  target="_blank">Cisco ACI Releases Changes in Behavior</a>
