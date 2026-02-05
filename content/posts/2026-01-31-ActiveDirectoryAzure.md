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

## Deploying Active Directory VM
Here is how i set up the VM, first specify the resource group and Subscription
Have all the resources on one region in my case EastUS
Specify the image to be Windows Server 2019 Datacenter Editon
You can look at the hardware requirements for the Windows Server : https://learn.microsoft.com/en-us/windows-server/get-started/hardware-requirements
For the disk, one OS disk for ISO, and later added more data disks.
And finally the networking, i created it 
![VM-p1](/images/2026-azure-VMp1.png)
![VM-p2](images/2026-azure-VMp2.png)
![VM-p3](images/2026-azure-VMp3.png)
![VM-p4](images/2026-azure-VMp4.png)