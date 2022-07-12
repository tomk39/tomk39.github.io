---
layout: post
title: How to run Cisco ACI Simulator on KVM host
categories: ACI
comments: true
---

<p align="center">
<img src="/images/acisim_on_kvm/subject.jpg" alt="Cisco ACI Simulator" title="Cisco ACI Simulator Spine leaf and APIC">  
</p>  
<p style='text-align: justify;'>
As officially ACI Simulator is not supported on KVM host and I didn't have ESXi in my lab I have decided to do experiment. 
Moreover I have played with amount of hardware resource necessary for this experiment. If you would like to know result of this experiment, you can proceed and read rest of the post.
</p>


<p style='text-align: justify;'>
At the beginning lets say couple of words about Cisco ACI Simulator or ACISIM. So this is Cisco virtual machine, inside which we can find simulation of two leaf switches, one spine and one Cisco Application Policy Infrastructure Controller (APIC). Actually this is simulation of Data Center leaf spine topology with SDN controller. 
</p>  

**<span style="color:#074080">Official Cisco ACI Simulator Virtual Machine HW requirements:</span>**  

<img style="center" src="/images/acisim_on_kvm/hw_resurces.jpg" alt="HW requirements" title="Official HW requirements">  
<p style='text-align: justify;'>
To make things more interesting I have reduced HW resources just from the beginning of experiment.&#128521;
</p>  

**<span style="color:#074080">And I created setup as below:</span>**  

<img style="float" width="350" height="100" src="/images/acisim_on_kvm/hw_resurces2.jpg" alt="HW requirements" title="HW requirements tested in the lab">  

<p style='text-align: justify;'>
So, are you ready to spin ACI Simulator with less than minimum HW resource for release 5.2(4d)? If yes, let's go and see what will happen in the end.&#128526;
</p> 
 **<span style="color:#074080">Step 1. Download ACI Simulator</span>**  

Go to <a href="https://software.cisco.com/download/home/286283149/type/286283168/release/5.2(4d)" target="_blank">Cisco Download Page</a>, please don't forget that you should have valid Cisco account, otherwise no luck.  

Download all **eight** pieces:
<ul>
<li>acisim-5.2-4d_part1.ova</li>
<li>acisim-5.2-4d_part2.ova</li>
<li>acisim-5.2-4d_part3.ova</li>
<li>acisim-5.2-4d_part4.ova</li>
<li>acisim-5.2-4d_part5.ova</li>
<li>acisim-5.2-4d_part6.ova</li>
<li>acisim-5.2-4d_part7.ova</li>
<li>acisim-5.2-4d_part8.ova</li>
</ul>
**<span style="color:#074080">Step 2. Concatenate downloaded parts and create QCOW2 image from VMDK</span>**  

Use Linux __cat__ command to concatenate all 8 parts.  

```
cat acisim-5.2-4d_part1.ova \ 
    acisim-5.2-4d_part2.ova \
    acisim-5.2-4d_part3.ova \
    acisim-5.2-4d_part4.ova \
    acisim-5.2-4d_part5.ova \
    acisim-5.2-4d_part6.ova \
    acisim-5.2-4d_part7.ova \
    acisim-5.2-4d_part8.ova > acisim.ova
```  

Optional check MD5 of acisim.ova VM once when concatenations is done.
```
md5sum -c aci.md5
```  

Now we have virtual machine with vmdk vHD but we need qocow2 image. So what's next, we need to untar acisim.ova and we have to convert acisim-5.2-4d-disk1.vmdk to acisim-5.2-4d-disk1.qcow2.   

```
tar -xvf aci.ova
```
voila finally we have acisim-5.2-4d-disk1.vmdk let's run one more command and we will have acisim-5.2-4d-disk1.qcow2
```
qemu-img convert -O qcow2 acisim-5.2-4d-disk1.vmdk path/to/images_folder/acisim-5.2-4d-disk1.qcow2
```

**<span style="color:#074080">Step 3. Run ACISIM VM on Ubuntu with KVM hypervisor</span>**  

<p style='text-align: justify;'>
Run virt-manger application from desktop UI or If you are using headless server first ssh to server (don't forget to enable x11 forwarding).
</p>  
<ul>
<li>Once when you run <b>virt-manager command</b> it will appear creation VM setup dialog</li>
</ul>

<img style="center" src="/images/acisim_on_kvm/Untitled0.jpg" alt="Virt-Manager1" title="Open VM setup dialog">  

<ul>
<li>Please choose like below on print screens.</li>
</ul>

<img style="float" width="350" height="275" src="/images/acisim_on_kvm/Untitled.jpg" alt="Virt-Manager2" title="Create new VM">  <img style="float" width="350" height="275" src="/images/acisim_on_kvm/Untitled 2.jpg" alt="Virt-Manager3" title="Choose qcow2 image">  

<ul>
<li>Here you will need to point to folder where you saved acisim.qcow2 image.</li>
</ul>

<img style="float" width="350" height="275" src="/images/acisim_on_kvm/Untitled 3.jpg" alt="Virt-Manager4" title="Choose qcow2 image"> <img style="float" width="350" height="275" src="/images/acisim_on_kvm/Untitled 4.jpg" alt="Virt-Manager5" title="Customize VM parameters">  

<ul>
<li>Let use only 8 vCPUs and 32G of RAM, this is far from recommended by Cisco. </li>
</ul>

<img style="float" width="350" height="275" src="/images/acisim_on_kvm/Untitled 5.jpg" alt="Virt-Manager6" title="Increase vCPU"> <img style="float" width="350" height="275" src="/images/acisim_on_kvm/Untitled 6.jpg" alt="Virt-Manager7" title="Increase Memory">  

<ul>
<li>Mandatory, please select network adapter type <b>e1000</b> otherwise later you will not be able to connect to your ACISIM via HTTPS or SSH.</li>
<li>To start VM creation press "Begin installation".</li>
</ul>

<img style="float" width="350" height="275" src="/images/acisim_on_kvm/Untitled 7.jpg" alt="Virt-Manager8" title="Set network"> <img style="float" width="350" height="275" src="/images/acisim_on_kvm/Untitled 9.jpg" alt="Virt-Manager9" title="Begin installation">  

<ul>
<li>Between all VMs choose acisim and run.</li>
</ul>

<img style="center" src="/images/acisim_on_kvm/Untitled 10.jpg" alt="Virt-Manager10" title="Select VM and Run">  

<ul>
<li>Press OPEN button to access acisim VM console.</li>
</ul>

<img style="center" src="/images/acisim_on_kvm/Untitled 11.jpg" alt="Virt-Manager11" title="Open Console">  

<ul>
<li>After 2-3 min (depend about power of your host) it will appear APIC initial screen.</li>
</ul>

<img style="center" src="/images/acisim_on_kvm/Untitled 12.jpg" alt="Virt-Manager12" title="APIC initial config">  

<ul>
<li>Below we can see that we are running Cisco ACI simulator on KVM host and successful start of initial configuration on Cisco APIC.</li>
</ul>

<img style="center" src="/images/acisim_on_kvm/Untitled 13.jpg" alt="Virt-Manager13" title="APIC initial config">  

**<span style="color:#074080">Short Outcome:</span>**  
<ul>
<li>If we have insufficient HW resource already in the lab we can reduce number of vCPUs and amount of memory
for ACISIM VM.</li> 
<li>In this Blog post I have used 8x vCPS and 32 GB of RAM below this it will not work.</li>
<li>We can use Cisco ACI simulator on KVM host, even if it is not officially supported by vendor.</li>
</ul>  
**<span style="color:#074080">Enjoy your ACI simulator and happy testing!!!</span>**  