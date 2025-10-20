---
layout: post
title: Five easy steps to configure static VxLAN Part 3 
categories: ["VxLAN"]
comments: true
---
<style>
p1 {
  font-size: 11px;
}
p2 {
  font-size: 12px;
}
</style>  
<p align="center">
<img src="/images/static_vxlan/vxlan_tunnel3.jpg" alt="Overlay VxLAN Cumulus Linux" title="VxLAN between Ubuntu and Cumulus VX">  
</p>
<p style='text-align: justify;'>
In Part 3 I will cover the case when Cumulus VX virtual switch is playing the role of DC edge gateway as well as interconnect between underlay and overlay network. On Cumulus virtual switch two VxLAN tunnels will be terminated from remote servers.  
</p>  

<p style='text-align: justify;'>
Details on how to download Cumulus Linux and VPCS (Virtual PC Simulator) can be found in 
<a href="/vxlan/static-vxlan-part2/index.html">Part 2</a>.
Like in Part 1 and 2, physical server with VMs will be deployed as virtual machine on baremetal KVM host and will be represented as "srv" and "srv1". Nested VM (application) will be simulated with Linux network namespace. On Cumulus VX virtual switch will be used bridge, which is not VLAN aware, as both tunnels from srv and srv1 will be terminated inside that bridge. Details on how everything is interconnected you can find below on the network diagram.
</p>
 

<p align="center"> 
  <img src="/images/static_vxlan/net_diag3.jpg" alt="VxLAN, VTEP, Overlay Network" title="VxLAN Network diagram"> 
</p>

**Prerequisites for servers (virtual machines):**

| VM name | OS Version | Extra installed SW |
| :--- |:--- | :--- |
| srv| Ubuntu 20.04 | bridge-utils |
| srv1| Ubuntu 20.04 | bridge-utils |
| virtual switch | Cumulus Linux 4.4.1 VX (Nvidia) | |

 **<span style="color:#074080">Step 1. Run VMs and install bridge-utils</span>**  

Let’s assume that virtual machine “srv” and "srv1" (Ubuntu), Cumulus VX vSwitch and VPCS are successfully deployed on KVM host.

Bridge-utils were installed on Ubuntu virtual machine using the command below:  

<code>apt install bridge-utils</code>  

version of bridge-utils we can check from CLI from srv and srv1 with the following command:  

<code>apt list | grep bridge-utils</code>

output should be

**<code>bridge-utils/focal,now 1.6-2ubuntu1 amd64 [installed]</code>**

version can vary.   

 **<span style="color:#074080">Step 2. Initial configuration of Linux srv, srv1  and Nvidia Cumulus VX virtual switch</span>**  

<u>Apply configuration on srv</u>  

Creation of namespaces.
```
ip netns add vm3 
```  
<u>Apply configuration on srv1.</u>
```
ip netns add vm1
```

Creation of bridges on both hosts srv and srv1.
```
ip link add br-vxlan type bridge
ip link set br-vxlan up
ip link set mtu 9000 dev br-vxlan
```  
Add static routes for Loopback IP on Cumulus VX switch on both hosts srv and srv1.
```
ip route add 192.168.1.4/32 via 192.168.100.24 dev ens0
```  
Disable spanning-tree on the bridges
```
brctl stp br-vxlan off
```
Creation of veth interfaces on both host VMs.

```
ip link add veth0 type veth peer veth1
ip link set up veth0
ip link set veth0 master br-vxlan
```  
<u>Apply configuration on srv.</u>  
Adding IP address on veth1 and bring interface up 
```
ip netns exec vm3 ip a a 192.168.22.1/24 dev veth1
ip netns exec vm3 ip link set up veth1
```  
<u>Apply configuration on srv1.</u>  
Adding IP address on veth1 and bring interface up.
```
ip netns exec vm1 ip a a 192.168.22.2/24 dev veth1
ip netns exec vm1 ip link set up veth1
```  
<u>Configure Cumulus VX virtual switch.</u>  

When you connect for the first time to Cumulus VX console it will request you to change default password.  

**<p1><u>Default User and Password for Cumulus VX:</u></p1>**
```
user: cumulus
pass: cumulus
```

Set hostname of Cumulus Linux virtual switch.
```
net add hostname leaf1
```  
Configure L3 interface for connection with underlay network.
```
net add interface swp1 ip address 192.168.100.24/24
```  
Configuration of Loopback interface, which will be used for VTEP termination point.
```
net add loopback lo ip address 192.168.1.4/32
```  
Add swp2 interface (please note interface will be used without VLAN id).
```
net add interface swp2 
```  
Create bridge not VLAN aware.
```
net add bridge br-12
```  
Add interface swp2 to bridge.
```
net add bridge br-12 ports swp2
```   

**<span style="color:#074080">Step 3. Configuration of  VxLAN Tunnel</span>**

<u>Apply on srv.</u>

Create VxLAN tunnel interface with VNI 101.
```
ip link add vx1 type vxlan id 101 local 192.168.100.20 remote 192.168.1.4 dev ens0 dstport 4789
```  
Bring up tunnel interface.
```  
ip link set vx1 up
```  
Add VxLAN tunnel interface to specific bridge.
```  
ip link set vx1 master br-vxlan
```  
<u>Apply on srv1.</u>

Create VxLAN tunnel interface with VNI 102.
```
ip link add vx1 type vxlan id 102 local 192.168.100.21 remote 192.168.1.4 dev ens0 dstport 4789
```  
Bring up tunnel interface.
```  
ip link set vx1 up
```  
Add VxLAN tunnel interface to specific bridge.
```  
ip link set vx1 master br-vxlan
```  
<u>Configure Cumulus VX virtual switch.</u>  

**<p1><u>Default User and Password for Cumulus VX:</u></p1>**
```
user: cumulus
pass: cumulus
```  
Add vni-101 and vni-102 interfaces.
```
net add vxlan vni-101 vxlan id 101
net add vxlan vni-102 vxlan id 102
```  
Add VxLAN interfaces to the bridge 
```
net add bridge br-12 ports vni-101
net add bridge br-12 ports vni-102
```  
For both tunnels set local and remote VETP address
```
net add vxlan vni-101 vxlan local-tunnelip 192.168.1.4
net add vxlan vni-101 vxlan remoteip 192.168.100.20

net add vxlan vni-102 vxlan local-tunnelip 192.168.1.4
net add vxlan vni-102 vxlan remoteip 192.168.100.21
```  

**<span style="color:#074080">Step 4. Add static MAC record to forwarding database</span>**  

<u>Apply on srv and srv1.</u>
```
bridge fdb append 00:00:00:00:00:00 dev vx1 dst 192.168.1.4
```  
This step we can skip on Cumulus Linux virtual switch as it will be applied automatically.

**<span style="color:#074080">Step 5. Connectivity check </span>**  

Run PING command from VM1, VM3 and DB server (VPCS)
<img style="center" src="/images/static_vxlan/con_check_vm1_vm3_db_cumulus.jpg" alt="Ping from namespace" title="Connectivity check with PING command">  

<u>Issue ping from VM3 (namespace)</u>
```
sudo ip netns exec vm3 ping 192.168.22.4 -- > remote IP on DB server.
```  

<img style="float" width="750" height="250" src="/images/static_vxlan/vm3_ping_db_part3.jpg" alt="L2 frame, VxLAN, wireshark, packet capture, pcap" title="Wireshark VxLAN packet capture" > 

<u>Issue ping from VM1 (namespace)</u>
```
sudo ip netns exec vm1 ping 192.168.22.4 -- > remote IP on DB server.
```  
<img style="float" width="750" height="250" src="/images/static_vxlan/vm1_ping_db.jpg" alt="L2 frame, VxLAN, wireshark, packet capture, pcap" title="Wireshark VxLAN packet capture" > 

From the capture you can see that two VxLAN tunnels have been used for interconnection with Cumulus VX virtual switch. Both tunnels landed on to one bridge on Cumulus VX virtual switch. That way we achieve communication between two VMs and DB server (VPCS).  

<u>Issue ping from DB server (VPCS) to check connectivity with VM1 and VM3</u>  

<img style="center" width="750" height="250" src="/images/static_vxlan/db_srv_ping_vm3_pcap_part3.jpg" alt="Ping from VPCS" title="Connectivity check with PING command">  

<br />
**<a href="/vxlan/static_vxlan_part1/index.html" target="_blank">Part 1 Static VxLAN between Ubuntu hosts</a>**  
**<a href="/vxlan/static-vxlan-part2/index.html" target="_blank">Part 2 Static VxLAN between Ubuntu and Cumulus VX vSwicth</a>**  
**<a href="/vxlan/static-vxlan-part4/index.html" target="_blank">Part 4 Static VxLAN Data Center Interconnect</a>**  
<br />