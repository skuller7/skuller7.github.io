Multi site deployment featuring Network security

The project will involve 3 components or domains of my knowledge that is : Operating systems, Cloud and mainly  Networking. 
Just to mention I will not be using any cloud provider in this project but, more cloud related architectures in Data centers and terms that are learned in cloud. 
The sole purpose of this project is to configure and create a multi site deployment environment using real Cisco images hosted on the Eve-ng machine and where the networking emulator is hosted on.
The project has 2 sites, Site A and Site B. 

Lets go deep into both sites and the connectivity between them. 

For site A i want to implement a Redundant and fault tolerant network design with multiple Switches and just a single Router.
Both networking infrastructure devices I got from Cisco CML which you do have to pay to obtain the license.
And we want to configure the following  : 
 - Configure hostname, Subnet Inital plan,
 - VLAN configuration that are divided into multiple Departments ( like HR, IT, DEV).
 - VLAN redunant trunking line to establish commnication from the same VLAN's 
 - Implemented EitherChannel Portchannel 0 configured with redundancy and Security. 
 - Added Inter Vlan Routing mechanism with Router on a Stick method.
 - ROAS has multiple sub interfaces for each VLAN departmant
 - Configured Dynamic NAT protcol with ACL rules
 - Connctivity between the Sites is eastablished via Static Routing

This ends and concludes the Site A configration, This is not a real representation of how firms actually work or some corporate network. Much more functinallity, automation, network programmability could be added.
This is from my view as a CCNA student on how would i implment a Network design.
For the Site B, i didnt really configure anything special, the Site B could act as fail over if a disaster stucts all the devices are connected just needs to be configured.

Here is the configuration and documentation for this mini project. 

1.	Configuring and adding VLAN’s on both switches 

VLAN or Virutal Local Area Networks are essential to corporate networks, they allow us to logically divide a network.They work on the Layer 2 of the OSI model

Switch>enable 
Switch#configure terminal
Enter configuration commands, one per line.  End with CNTL/Z.
Switch(config)#hostname SW1 // Configure the name for the switch
SW1(config)#vlan 10 // Adding vlans command 
SW1(config-vlan)#name IT // naming them is optional but not required
SW1(config-vlan)#exit
SW1(config)#interface gig
SW1(config)#interface gigabitEthernet 1/0
SW1(config-if)#switchport mode access  
SW1(config-if)#switchport access vlan 10 
SW1(config-if)#exit
