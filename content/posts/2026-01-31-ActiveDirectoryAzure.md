## Active Directiry Lab In Azure
This was an mini project i did for my University class 
In this project, i set up a simple network & cloud architecture
The Network architecture is called Hub and Spoke.
Its core design was Centralization of Network traffic, scalability and security.

### Project Architecture Overview
Inside the Hub vNet was the Windows Server 2019 DC VM and on the spoke vNet were Windows 10 clients VM.
An Azure P2S VPN was configured to access the internal network.
Configured and Set up Active Directory, added the Clients to the Domain. 
Implemented Policy specific on clients VM’s

### Active Direcory & Network plan

For this project, i simulated a typcal AD DS Lab with 3 departmants, HR SALES IT.
I set up different group policies based on departments.
Meaning that Clinets that are part of SALES dep have different policies as for someone that is in IT dep.  

MAIN - Hub: Network 10.0.1.0 - 10.0.1.255 Subnet --> 24 | Active Directory Server 10.0.1.4
HR - Spoke A: Sophie | Network  10.1.1.0 - 10.1.1.255 Subnet --> 24 | Windows 10 10.1.1.5
SALES - Spoke B: Mary | Network 10.2.1.0 - 10.2.1.255 Subnet --> 24 | Windows 10 10.2.1.6
IT - Spoke C: Uros | Network 10.3.1.0 - 10.3.1.255 Subnet --> 24 | Windows 10 10.3.1.7

#### Virtual Machine deployment

Here is how i set up the VM, first specify the resource group and Subscription
Have all the resources on one region in my case EastUS
Specify the image to be Windows Server 2019 Datacenter Editon
You can look at the hardware requirements for the Windows Server : https://learn.microsoft.com/en-us/windows-server/get-started/hardware-requirements
For the disk, one OS disk for ISO, and later added more data disks.
And finally the networking, i created in the background all the 4 networks just like we specified above.
![VM-p1](/images/2026-azure-VMp1.png)
![VM-p2](/images/2026-azure-VMp2.png)
![VM-p3](/images/2026-azure-VMp3.png)
![VM-p4](/images/2026-azure-VMp4.png)
![VM-p5](/images/2026-azure-VMp5.png)
After you create the VM, we can log in via RDP make sure to open port 3306 so that you can connect.
Now that we Logged in, lets set the static IP, we can do that by
Going to control panel => Network and Sharing center => Select Ethernet => Properties => And select TCP/IPv4 click on properties => Use the following ip address And enter these parameters like in the image.
![VM-p6](/images/2026-azure-VMp6.png)

Before you install the Active Directory, lets configure the 2 data disks 
If you didnt already add them, you can do so on Azure click on your VM => Disks i created 2 actually
![VM-p7](/images/2026-azure-VMp7.png)
![VM-p8](/images/2026-azure-VMp8.png)
So now we need to format mount them and set up FS.
![VM-p9](/images/2026-azure-VMp9.png)

#### Active Directory Installation

Now lets set up AD, Click on the Add roles and Features
Select Active Directory Domain Services only, and leave everything on by default
![VM-p10](/images/2026-azure-VMp10.png)
Lets start the configuration of Active Directory
here we will create a new forest for our Domain, be sure to select DNS server also.
I used lilixkube as my domain, you can create your own. 
![VM-p11](/images/2026-azure-VM11.png)
![VM-p12](/images/2026-azure-VM12.png)

Now that we set up the Active Directory VM, lets focus on the client workstations.
If you didnt create networking on the azure do it so by going to vnet this is an example for Spoke A 
![VM-p13](/images/2026-azure-VMp13.png)
![VM-p14](/images/2026-azure-VMp14.png)
After you set up all spoke virtual networks, you can see them here
![VM-p15](/images/2026-azure-VMp15.png)

#### Virtual Network Peering
Now it is necessary to establish vnet-peering, it is an integrated mechanism within vNet that allows us to connect several virtual networks and ensure connectivity within the subnet.
In Hub and spoke architecture, We connect all tree spoke networks with a Hub network.
Also, later we will change the option for the Gateway it applies to the VPN, which we will configure later.
