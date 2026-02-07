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
![VM-p11](/images/2026-azure-VMp11.png)
![VM-p12](/images/2026-azure-VMp12.png)

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
Be sure to repeat the same steps for Hub to Spoke networks and vice versa
![VM-p16](/images/2026-azure-VMp16.png)
![VM-p17](/images/2026-azure-VMp17.png)
![VM-p18](/images/2026-azure-VMp18.png)

#### VPN PointToSite Configuration
Now that we created networks and configured peering.
We need to set up a mechanism where we can connect to our azure internal network.
One of the ways is through VPN, using P2S model.
This is possbile with Azure VPN gateway 

1. First thing that we need is create an extra subnet for VPN gateway
This is actually needed when we connect to the VPN, the OS creates an interface for VPN with that address pool of 10.10.0.0/24 
![VM-p19](/images/2026-azure-VMp19.png)
![VM-p20](/images/2026-azure-VMp20.png)

2. Create & set up VPN gateway
Set the VPN to attach to Hub network
Be sure to watch for the VPN sku i used no availability gen 1, check pricing here: https://azure.microsoft.com/en-us/pricing/details/vpn-gateway
Deployment can take up to 30 mins.
![VM-p21](/images/2026-azure-VMp21.png)
![VM-p22](/images/2026-azure-VMp22.png)

3. Setting up Root Certificates
Here is the link for the command to create an certificate : https://learn.microsoft.com/en-us/azure/vpn-gateway/vpn-gateway-certificates-point-to-site
For windows based systems open Powershell as Admin and paste this in
```sh
$params = @{
    Type = 'Custom'
    Subject = 'CN=P2SRootCert'
    KeySpec = 'Signature'
    KeyExportPolicy = 'Exportable'
    KeyUsage = 'CertSign'
    KeyUsageProperty = 'Sign'
    KeyLength = 2048
    HashAlgorithm = 'sha256'
    NotAfter = (Get-Date).AddMonths(24)
    CertStoreLocation = 'Cert:\CurrentUser\My'
}
$cert = New-SelfSignedCertificate @params
```
You can find the created certificate through Certificate Manager program On Windows
Click on Certificates and check if your name is there in my case P2SRootCert
When you find it, right click => All tasks => Export => Do not export Privatekey => Option base64 => Save it in a file on your FS.
![VM-p23](/images/2026-azure-VMp23.png)

4. Copy Certificate to Azure & Download .vpn file
Lets copy our Certificate that we just saved and in Azure go to Hub-VPN-gateway Point to Site Configuration
![VM-p24](/images/2026-azure-VMp24.png)
When you do so, download the zip file, extract it find the file VPNsetting.xml
![VM-p25](/images/2026-azure-VMp25.png)

5. Connecting to the VPN
Open the VPN setting on Windows, Add VPN
Copy from the VPNsettings.xml file VPNserver name id, paste it as a Server Name
For the type set it as Certificate and protocol IKEv2
Once done so you can Connect to the VPN.
![VM-p26](/images/2026-azure-VMp26.png)

#### Setting up Windows Client 
Now that we have VPN connection, lets set up an VM Client and test connectivity
In azure quickly deploy this VM, just set the network to one of the Spoke, set the public IP to none
![VM-p27](/images/2026-azure-VMp27.png)  
Now lets first connect to the Windows Server from HUB net and try to test connectivity to the newly created Mary Client Machine.
I used the default remote desktop connection, you can use any software you want.
If you notice we can see We entered the private IP address of the Windows Server VM, and its username. We get an active directory domain name pop up, so that is great news.   
![VM-p28](/images/2026-azure-VMp28.png)
Now from our WIN server machine lets test connectivity to Mary client VM and connect to the VM.
![VM-p29](/images/2026-azure-VMp29.png)
![VM-p30](/images/2026-azure-VMp30.png)
That is basically the procedure on how we connect to the Client

#### Connecting the User to the Domain
Right now the problem is that that client is not part of the domain name.
Lets connect to the domain for a different User, in my case Uros from IT here is how we can do that.
There are multiple things we need to set up
First thing from client side join the domain (THIS WONT SOLVE OUR PROBLEM)
![VM-p37](/images/2026-azure-VMp37.png)
You can see that by actually trying to connecto that client after applying the above changes
![VM-p38](/images/2026-azure-VMp38.png)
Here is how to FIX This Issue 
1. On windows server, go to Active directory Users and Computers => create an User
![VM-p31](/images/2026-azure-VMp31.png)
2. Set up An OU 
Go to Users and Computers => New OU => Move the User (Uros) to the New OU
Right click on the Uros and click on Move to 
3. Group Policy Management  
Open and create a new GPO directly on that OU
![VM-p32](/images/2026-azure-VMp32.png)
4. Set up GPO
Now lets configure the policy, by clicking on the edit
Computer Configuration --> Policies --> Administrative Templates --> Windows Components --> Remote Desktop Services --> Remote Desktop Session Host --> Connections
Computer Configuration --> Windows Settings --> Security Settings --> Local Policies --> User Rights Assignment
![VM-p33](/images/2026-azure-VMp33.png)
![VM-p34](/images/2026-azure-VMp34.png)
5. Connecting to the Domain Client
To connect to the client we need to use this format : DOMAIN_NAME\CLIENT
In addition to that i implemented an policy that deny access to some programs
![VM-p35](/images/2026-azure-VMp35.png)
![VM-P36](/images/2026-azure-VMp36.png)