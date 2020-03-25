## Dnsmasq envirznment setup
### The environment 

IoTGoat’s virtual machine is connected to the Host via NAT network by default as shown in the image below. The Host Machine provides an IP via DHCP to IoTGoat. Thus, the network connection including internet access to the VM is provided by the Host Machine. 

![Fig 1](https://raw.githubusercontent.com/paragmhatre1993/IoTGoat/master/images/enviornment1.png)


To perform the dnsmasq memory corruption attack, we will have to exploit the DHCPv6 service of dnsmasq. Thus, we have to enable IoTGoat to run the DHCP service(give out IP addresses). 

We will create a custom subnet(network) to simulate LAN for IoTGoat. IoTGoat will run the dnsmasq DHCP service and give out IP addresses to other VMs connected on that custom network and the custom network of the Host Machine. 

![Fig 2](https://raw.githubusercontent.com/paragmhatre1993/IoTGoat/master/images/enviornment2.png)
**IoTGoat in router configuration VS Home router configuration**

A router is also connected to the internet(WAN). Similarly, the Host Machine will be connected via NAT network to the router’s WAN network device. Thus, the Host Machine will also act as the ISP(Internet Service Provider) for IoTGoat. 

![Fig 3](https://raw.githubusercontent.com/paragmhatre1993/IoTGoat/master/images/enviornment3.png)

To do the above configuration, we have to create a custom network to simulate LAN for IoTGoat. Other virtual machines can be connected to this network by choosing that network for the Network device connected to that VM. The default IP address for IoTGoat is 192.168.99.1

### Host setup (Create a custom subnet)

1. Create a custom network.
2. Add 2 Network adapters to IoTGoat.

On your virtualization software(VMWare, Virtualbox, etc.), you need to create a custom network that can be used by IoTGoat as its LAN interface. The network should have the following specifications:

1. You can choose to connect the Host Machine to this subnet or not. We recommend not connecting it as we have faced issues with this configuration.
2. Disable DHCP for this subnet(We will use IoTGoat’s DHCP service).

A few helpful links to help you configure this on some popular platforms.

VMWare Fusion  
[https://pubs.vmware.com/fusion-5/index.jsp?topic=%2Fcom.vmware.fusion.help.doc%2FGUID-C5837B81-9509-4F1B-9572-0EC0CFA87563.html](https://pubs.vmware.com/fusion-5/index.jsp?topic=%2Fcom.vmware.fusion.help.doc%2FGUID-C5837B81-9509-4F1B-9572-0EC0CFA87563.html) 

![Fig 4](https://raw.githubusercontent.com/paragmhatre1993/IoTGoat/master/images/fusion_network.png)

VMWare Workstation  
[https://docs.vmware.com/en/VMware-Workstation-Pro/12.0/com.vmware.ws.using.doc/GUID-E08995A9-7DE6-4D92-8C6F-4F737C3A8AB3.html](https://docs.vmware.com/en/VMware-Workstation-Pro/12.0/com.vmware.ws.using.doc/GUID-E08995A9-7DE6-4D92-8C6F-4F737C3A8AB3.html)

![Fig 5](https://raw.githubusercontent.com/paragmhatre1993/IoTGoat/master/images/workstation_network.PNG)


VirtualBox  
[https://www.techrepublic.com/article/how-to-create-multiple-nat-networks-in-virtualbox/](https://www.techrepublic.com/article/how-to-create-multiple-nat-networks-in-virtualbox) 

![Fig 6](https://raw.githubusercontent.com/paragmhatre1993/IoTGoat/master/images/virtualbox_network.PNG)

### Attacker setup

The attacker computer should have a Network adapter that is connected to the new custom interface. This machine should get an IPv4 from IoTGoat which by default should be in the 192.168.99.0/24 subnet and an IPv6 address as shown below.

![Fig 7](https://raw.githubusercontent.com/paragmhatre1993/IoTGoat/master/images/attacker_ifconfig.png)

### IoTGoat setup

Two Network adapters must be created within the IoTGoat virtual machine (Select the “Add Device” and choose the “Network Adapter” device to add). 

The first network adapter is supposed to be connected to the custom network that we just created (vmnet6 in the image above). 

The second network adapter must be connected to Host VM via NAT. 

![Fig 8](https://raw.githubusercontent.com/paragmhatre1993/IoTGoat/master/images/iotgoat_network.png)

Now we need to make some changes in the network configuration. We should assign static IPs to IoTGoat and configure dnsmasq to give out IPv4 and IPv6 addresses in specific subnets via DHCP. 

We have created a script for performing the above changes. You can execute the script from the root folder as shown below. If you want to revert back to the original settings, you can just execute it with “-d” flag.


```
$ /dnsmasq_netconfig.sh 
```

### Launching the attack

Attack instructions are noted in Google’s security research PoCs repository [https://github.com/google/security-research-pocs/blob/master/vulnerabilities/dnsmasq/CVE-2017-14493-instructions.txt](https://github.com/google/security-research-pocs/blob/master/vulnerabilities/dnsmasq/CVE-2017-14493-instructions.txt). For simplicity, download the raw python script for CVE-2017-14493 via wget and execute the payload targeting IoTGoats statically configured IPv6 address at port 547 as shown below. 


```
$ wget https://raw.githubusercontent.com/google/security-research-pocs/master/vulnerabilities/dnsmasq/CVE-2017-14493.py

$ python CVE-2017-14493.py fdca:1:2:3:4::1234 547
```

![Fig 9](https://raw.githubusercontent.com/paragmhatre1993/IoTGoat/master/images/attacker_script.png)

The following image shows the crash behavior from IoTGoat. 

![Fig 10](https://raw.githubusercontent.com/paragmhatre1993/IoTGoat/master/images/segmentation_fault.png)

