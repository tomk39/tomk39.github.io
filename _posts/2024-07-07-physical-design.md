---
layout: post
title: Mastering Cisco ACI Physical Design
categories: ["Cisco ACI"]
---

<p align="center">
<img src="/images/phy_design/2.png" alt="Switch SFP Cables" title="ACI Phy Design"> 
</p>
<p style='text-align: justify;'>
Let's dive into the world of Cisco ACI physical design. After all, we all know that without hardware, there is no cloud or networking.
</p>

<p style='text-align: justify;'>
Dive into the world of Clos Topology, a concept born from a 1952 paper by Charles Clos, a researcher at Bell Laboratories. His revolutionary ideas on multistage telephone switching systems have transcended time, now serving as the foundation for building highly scalable data centers with cost-effective switches.
The Cisco ACI fabric is based on either a two-tier (spine and leaf switch) or three-tier (spine switch, tier-1 leaf switch, and tier-2 leaf switch) architecture.
In a Cisco Application Centric Infrastructure (ACI), each of the leaf, Fabric Extenders and spine switches has different functions.
</p>

**<span style="color:#074080">Leaf Switches</span>** 

- Leaf switches are the ingress/egress points for traffic into and out of an ACI fabric.
- They connect directly to servers and other devices, facilitating rapid data transfer.
- Each leaf switch is linked to every spine switch, creating a mesh of high-bandwidth connections.
- This setup ensures that any data packet is only a couple of hops away from its destination, significantly reducing latency.

**<span style="color:#074080">Fabric Extenders (FEX)</span>** 

Fabric Extenders (FEX) in Cisco's Application Centric Infrastructure (ACI) have several key functions:
- Port Extension - FEX extends the ports of a Nexus switch, acting as a remote line card⁶. This allows for the use of more cost-effective devices with a reduced set of features that are connected to the Leaf Switches.  
- Agile Connectivity - FEX offers agile connectivity for rack and blade servers and for converged fabric deployments⁵. It delivers innovation to the data center, reduces total cost of ownership, and gains architectural flexibility.
- Configuration and Management - All management is done on the parent switch; there is no console/vty access on the FEX⁶. This simplifies the management of the network infrastructure.
- Cisco ACI leaf switch models with -G or -Tin the product name, such as Cisco Nexus N9K-C9348GC-FXP, N9K-C93108TC-FX, N9K-C93108TC-FX-24, N9K-C93108TC-EX, N9K-C93108TC-EX-24, N9K-C93216TC-FX2, and N9K-93108TC-FX3P.  

Discover the steps to configure Fabric Extenders (FEX) in Cisco ACI in Cisco comprehensive guide available <a href="https://www.cisco.com/c/en/us/support/docs/cloud-systems-management/application-policy-infrastructure-controller-apic/200529-Configure-a-Fabric-Extender-with-Applica.html"  target="_blank">here</a>.


**<span style="color:#074080">Spine Switches</span>** 

Spine switches are Clos intermediary switches that have a number of key functions:  

- They are responsible for high-capacity data transmission across the network.
- They ensure that data packets travel efficiently from one leaf switch to another.
- The spine layer is crucial for maintaining the performance of the network as it scales, providing a robust framework that can handle increasing data demands without bottlenecks.
- Spine switches interconnect all leaf switches in a full-mesh topology.

Together, they create an architecture that is highly standardized across deployments. This design allows for a highly scalable and efficient network infrastructure.


##### **<span style="color:#074080">Two-Tier or Multi-Tier Design?</span>** 

<p style='text-align: justify;'>
Below Figure 1 illustrates the necessary components and cabling for an two-tier ACI fabric. It’s important to note that no cables should be connected between ACI leaf switches, as this is characteristic of Clos network design. Similarly, cross-cabling ACI spines will lead to the disabling of the cross-connected ports. The topology depicts a full mesh of cabling between the spine and leaf layers. 
However, it’s important to note that while the Spine-Leaf architecture can be implemented without ACI, with the switches working in NX-OS mode. I would like to highlight that ASICs are different for Leaf and Spine switches, so a switch like the Nexus N9K-93180YC-EX cannot run as Spine switches.  
</p>

<p align="center">
<img class="center" style="float" width="350" height="350" src="/images/phy_design/aci-PhysicalDesign.jpg" alt="phy-design" title="PHY">  
</p>


**Figure 1** Two-Tier Topology Spine-Leaf classic interconnect between leaf and spine switches


In summary, I would like to recommend to check switch compatibility on the links below:


<ul>
<li><a href="https://www.cisco.com/c/dam/en/us/products/switches/nexus-9000-series-switches/nexus-9300-10GbaseT-switches-comparison.html"  target="_blank">Cisco Nexus 9300 1/10GBaseT Switches</a></li>
<li><a href="https://www.cisco.com/c/en/us/products/switches/nexus-9000-series-switches/nexus-9300-10ge-fiber-switches-comparison.html"  target="_blank">Cisco Nexus 9300 1/10/25/40/50/100GE Fiber Switches</a></li>
<li><a href="https://www.cisco.com/c/dam/en/us/products/switches/nexus-9000-series-switches/nexus-9300-40GE-switches-comparison.html"  target="_blank">Cisco Nexus 9300 40/100 GE Switches</a></li>
<li><a href="https://www.cisco.com/c/en/us/products/switches/nexus-9000-series-switches/nexus-9300-400-ge-switches.html"  target="_blank">Cisco Nexus 9300 400 GE Switches</a></li>
<li><a href="https://www.cisco.com/c/en/us/products/collateral/switches/nexus-9000-series-switches/datasheet-c78-731792.html"  target="_blank"> Cisco Nexus 9300 ACI Fixed Spine Switches Data Sheet</a></li>
</ul>

If you plan to use Cisco Nexus modular switches for Cisco ACI fabric technical details you can find on below links:

- <a href="https://www.cisco.com/c/en/us/products/collateral/switches/nexus-9000-series-switches/datasheet-c78-729404.html"  target="_blank"> Cisco Nexus 9500 Series Switches Data Sheet</a> 
- <a href="https://www.cisco.com/c/en/us/products/switches/nexus-9000-series-switches/nexus-9500-chassis-comparison.html"  target="_blank"> Nexus 9500 Chassis Comparison</a>
- <a href="https://www.cisco.com/c/en/us/products/switches/nexus-9000-series-switches/nexus-9800-models-comparison.html"  target="_blank"> Nexus 9800 Compare Models</a>
- <a href="https://www.cisco.com/c/en/us/products/switches/nexus-9000-series-switches/nexus-9800-line-cards-comparison.html"  target="_blank"> Nexus 9800 Line Cards comparison</a>


<p style='text-align: justify;'>
Starting from Cisco ACI 4.1, the fabric also supports a multi-tier (three-tiers) topology. This includes two tiers of leaf switches, allowing for vertical expansion of the Cisco ACI fabric. This is particularly useful for migrating a traditional three-tier architecture (core-aggregation-access), a common design model for many enterprise networks.
The ACI Multi-Tier is a beneficial solution for data centers aiming to reduce inter-row cabling and have low-bandwidth needs for top-of-rack switches. In this setup, Tier 1 leaf switches are placed at the end of each row, while Tier 2 leaf switches are positioned at the top of each rack. This strategy is particularly effective in managing cables and meeting bandwidth requirements.
</p>

<p align="center">
<img class="center" style="float" width="450" height="400" src="/images/phy_design/aci-PhysicalDesign 2.jpg" alt="phy-design" title="PHY">  
</p>


**Figure 2** Multi -Tier Topology Spine- Tier 1 Leaf and Tier 2 Leaf


<p align="center">
<img class="center" style="float" width="400" height="300" src="/images/phy_design/aci-Sec PhysicalDesign 2.jpg" alt="phy-design" title="PHY">  
</p>  


**Figure 3** Multi -Tier Topology Spine- Tier 1 Leaf and Tier 2 Leaf option between two data center buildings


The topology requirements may include various models of spine and leaf switches, please check below:

- **Spine:** EX/FX/C/GX/GX2 spines (For example, Cisco Nexus 9332C, 9316D-GX, 9348D-GX2A, 9408 and 9500 with EX/FX/GX linecards).

- **Tier-1** Leaf: EX/FX/FX2/FX3/GX/GX2 except Cisco Nexus 93180LC-EX.

- **Tier-2** All EX/FX/FX2/FX3/GX/GX2.


In the scenario where server access is at 1G and there is not need for high bandwidth, it’s a viable choice to use the Cisco Nexus 93180FX3 or Nexus 9364C-GX as the tier-1 leaf and the Cisco Nexus 9348FXP as the tier-2 leaf. 


Before you start to build and design a multi-tier topology in a Cisco ACI environment, please take into account the following:


**<span style="color:#074080">1.Fabric Port Configuration:</span>** All inter-switch connections must be configured as fabric ports. This includes the connections between Tier-2 leaf switch fabric ports and Tier-1 leaf switch fabric ports.


**<span style="color:#074080">2.Tier-2 Leaf Switch Connectivity:</span>** A Tier-2 leaf switch has the capability to connect to more than two Tier-1 leaf switches. This is in contrast to a traditional double-sided vPC design, which only supports two upstream switches. The maximum number of ECMP links supported by a Tier-2 leaf switch to a Tier-1 leaf switch is 18.


**<span style="color:#074080">3.Endpoint Group (EPG) and Layer 3 Outside (L3Out) Connections:</span>** EPGs, L3Outs, Cisco APICs, or Fabric Extenders (FEX) can be connected to either Tier-1 or Tier-2 leaf switches.


**<span style="color:#074080">4.Tier-1 Leaf Switch Connections:</span>** Tier-1 leaf switches can have both hosts and Tier-2 leaf switches connected to them.


**<span style="color:#074080">5.Switch Tier Transition:</span>** Transitioning from a Tier-1 to a Tier-2 leaf switch (and vice versa) requires decommissioning and recommissioning the switch.


**<span style="color:#074080">6.Compatibility with Cisco ACI Multi-Pod and Multi-Site:</span>** Multi-tier architectures are compatible with both Cisco ACI Multi-Pod and Cisco ACI Multi-Site.


**<span style="color:#074080">7.Tier-2 Leaf Switch Connectivity Limitations:</span>** Tier-2 leaf switches cannot be connected to remote leaf switches (Tier-1 leaf switches).


**<span style="color:#074080">8.Scale Considerations:</span>** The combined number of Tier-1 and Tier-2 leaf switches must not exceed the maximum number of leaf switches validated for a given release (400 per pod; 500 per Cisco ACI Multi-Pod as of Cisco ACI release 6.0(1)).

Reference for <a href="https://www.cisco.com/c/en/us/products/switches/nexus-9000-series-switches/nexus-9800-line-cards-comparison.html"  target="_blank"> Cisco ACI Multi-tier Architecture</a>.

***<span style="color:#074080">Fabric Port</span>***

<p style='text-align: justify;'>
In Cisco's Application Centric Infrastructure (ACI), a fabric port is a type of port that is used for uplink connectivity within the fabric. These ports are typically used to connect different switches within the ACI fabric.
For example, in a multi-tier topology, all switch-to-switch links must be configured as fabric ports. This includes the connections between Tier-2 leaf switch fabric ports and Tier-1 leaf switch fabric ports.
</p>

**<span style="color:#074080">Fiber optics modules</span>** 

<p style='text-align: justify;'>  
Don't forget that based on specific switch model and uplinks design you will need to choose proper optical modules.
All details about compatibility matrix for Cisco Optics and devices you can find on below links:
</p>
- <a href="https://tmgmatrix.cisco.com/">Cisco Optics-to-Device Compatibility Matrix</a>  
- <a href="https://tmgmatrix.cisco.com/iop">Cisco Optics-to-Optics Interoperability Matrix</a>  
<p style='text-align: justify;'> 
If you struggling or you have some doubts about compatibility between switches and optical modules SFP SFP+ QSFPs I would like to recommend:
</p>

<p>
<iframe width="420" height="315"
src="https://www.youtube.com/embed/5I19ePqzNhc?controls=0">
</iframe>
</p>

<p>
<iframe width="420" height="315"
src="https://www.youtube.com/embed/FFhNGxmniR4?controls=0">
</iframe>  
</p>

**<span style="color:#074080">Taxonomy for Cisco Nexus 9000 Series Part Numbers</span>**  

<p style='text-align: justify;'>
For a comprehensive understanding and decoding of all Nexus Part Numbers names, kindly refer to Figure 4.
</p>


<p align="center">
<img class="center" style="float" width="800" height="400" src="/images/phy_design/taxonomy_nk9.png" alt="taxonomynk9" title="PHY">  
</p>

**Figure 4** Cheet-sheet for decoding Cisco Nexus 9000 Series Part Numbers

