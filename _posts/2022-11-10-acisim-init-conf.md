---
layout: post
title: Cisco ACI Simulator APIC Initial setup
categories: ACI
comments: true
---
<p align="center">
<img src="/images/acisim_init_conf/subject.jpg" alt="Cisco ACI Simulator" title="Cisco ACI Initial Configuration">  
</p>  
<p style='text-align: justify;'>
If you already read my last post <a href="/aci/acisim-on-kvm/index.html" target="_blank">How to run Cisco ACI Simulator on KVM host</a> you know that now we need to do first step and go through Cisco APIC initial configuration setup. If you need help this post is right place for you. So let' go and see what we need to do.
</p>  


<img class="center" style="float" width="400" height="400" src="/images/acisim_init_conf/net_diagram.jpg" alt="ACISIM Diagram" title="ACISIM internal connectivity">  

```
Note:
Cisco ACI simulator doesn't support data plane and you won't be able to connect any  
other VM which can represent End Point. Simple you will not be able to issue ping  
command and send ICMP request and see any ICMP reply or use any other protocol for testing purpose.
```

1.VM is booting
<img class="center" width="800" height="400" src="/images/acisim_init_conf/1.jpg" alt="acisim booting" title="acisim booting">  
  
  
2.If you have recommended HW resource use large topology otherwise your choice is "N".
<img class="center" width="800" height="280" src="/images/acisim_init_conf/2.jpg" alt="Acisim topology size" title="Acisim topology size">  

3.After few seconds it will appear this nice logo.
<img class="center" width="800" height="280" src="/images/acisim_init_conf/3.jpg" alt="APIC Welcome screen" title="APIC Welcome screen">  

4.For ACISIM I would like to recommend to accept all default parameters. All parameters you can change but in that case there is no guaranteed that Cisco ACI simulator will work properly.
<img class="center" width="800" height="s 350" src="/images/acisim_init_conf/7.jpg" alt="APIC ACISIM setup" title="APIC ACISIM setup">  

5.One of parameters what actually you should change is default OOB subnet and gateway. Choose IP address/Subnet mask from your OOB lab network and set in this step. Later you will use this IP address for access APIC UI via HTTPS.
<img class="center" width="600" height="100" src="/images/acisim_init_conf/8.jpg" alt="APIC ACISIM OOB" title="APIC ACISIM OOB">  

6.Please set admin password and don't forget, later you will need for login to APIC UI &#128521;
<img class="center" width="600" height="100" src="/images/acisim_init_conf/9.jpg" alt="APIC ACISIM admin pass" title="APIC ACISIM admin pass">  

7.Here you have chance to change any configuration parameter what you set during initial configuration.
If you are self confident and you put password on the sticky note on your desk you just press "ENTER"
<img class="center" width="600" height="120" src="/images/acisim_init_conf/10.jpg" alt="APIC ACISIM modification" title="APIC ACISIM modification">  

8.In your favorite browser type https://oob.ip.address what you set during initial configuration.
When APIC login page appear for User ID use **admin** and for password use note from your sticky note from your the desk.
<img class="center" width="800" height="400" src="/images/acisim_init_conf/login_screen1.jpg" alt="Login" title="Login">  


Final Part 2 coming soon! Are you enjoyed reading? If yes please subscribe **<a href="/subscribe/index.html" target="_blank">Here</a>**


<head>
<style>
a:link {
  text-decoration: underline;
}

a:visited {
  text-decoration: none;
}

a:hover {
  text-decoration: none;
}

a:active {
  text-decoration: none;
}
.center {
  display: block;
  margin-left: auto;
  margin-right: auto;
}
</style>
</head>


