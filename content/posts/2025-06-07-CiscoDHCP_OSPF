## Cisco EVE-NG  - DHCP & OSPF Lab

![Network Topology](/images/OSPF-DHCP-Cisco-topology.png)

So the main idea was creating a OSPF Enterprise network with 5 routers in total.
We can see that there are R1,R2,R3 and R4, and there is R5 that is separated by the switch. 
On the right side of the picture I used a simple ip addressing scheme, so it tells us which subnet is configured on which node. 
Every OSPF Router is advertising its directly connected network in the form of LSA’s  message that is later sent as a Multicast message to other Neigboring Routers.
The R5 acts as a DHCP Server and has multiple pools for each internal/LOCAL Router Network. This is done by configuring DHCP Relays where the VPCS are located and acts as a forwarder

## OSPF info and configuration

So we all know it as a Routing protocol that allows us automatically learn all the routes on an network architecture.
OSPF is an link state protocol and works inside a local network rather than something like BGP.
OSPF learns these routes by sharing its directly connected interfaces information like its ip address duplex speed is it up or not description, and all that data or payload is grouped and called an LSA.
LSA's are transmitted to all of the routers in an network architecture so that all of the routers can get them and keep them 
Once every router has basically picture of every Router node and how it is connected, the shortest path first method is implemented.
The one thing that OSPF stands out other than other Routing protocols is its metric COST. Here is an example. if we have a GigEth interface that is like 40gigs and a GigEth interface of 1 gig, By default they have the same COST value of 1 that is not good in some DC where speed is madatory. But with COST addItionally confgiured on the Router we can make the 40 Gig interface have Lower the cost value and make sure that our data is routing only through that interface. 

Here is how i configured OSPF : 

![OSPF Config](/static/images/slika-OSPF.png)

The madatory configuration that is needed is specifying the process id and networks.
For the network part you should specify the networks you are directly connected to.

```cisco
enable
configure terminal
Interface Loopback 0
Ip address 2.2.2.2
exit 
Router ospf 1 // Process ID of OPSF can be anything 
Network 10.12.0.0 0.0.0.15 area 0 // this in the end is wildcard mask more
Network 10.13.0.0 0.0.0.15 area 0 
Network 10.24.0.0 0.0.0.15 area 0
```

Quick tip on caluculating the wildcard mask is like this : 255.255.255.255 - 255.255.255.240 (Ur Subnet mask) = 0.0.0.15

Once you do for every router use this command on the image to view adjacent neighbors
Also this image proves that we can ping the DHCP Server and we can reach that node by sending a ping.
![OSPF Neighbors](/static/images/OSPF-neighbors.png)

## DHCP Configuration

DHCP is a Networking protocol that is responsible for automatically assigning private ip addresses to DHCP clients.
It is located on the L7 and works on UDP protcol 67,68.
DHCP has a simple process on how it assigns ip address to clients. 
That process is called DORA - Disover Offer Offer Ack. 
 - The Disover message is a brodcast message coming from the client requesting an ip address (L2 MAC Brodcast)-FF:FF:FF
 - Reuest reaches the DHCP Server and looks at his pool of avaiable addresses and offers it to the client by sending a Unicast.
 - Client responds to the Server saying i chose this address and want to use it - sends a brodcast
 - Finally the server ackowledges the client request and confirming the requested IP. - Unicast

### DHCP Server

For the DHCP Server to assign ip addresses you need a DHCP Pool i created multiple pools for each client VPC
It is also important to specify the default gateway for the DHCP client and NOT exclude that address.

![DHCP Server](/static/images/slika-DHCP-2.png)

```cisco
enable
configure terminal
ip dhcp pool IT-DHCP-Pool
network 172.31.10.0 255.255.255.0
default-router 172.31.10.99
end
```
### DHCP Relay Agents

Because i have multiple networks and the DHCP Server is not on same subnet as the Clients are we need a Relay agent
That role is typically played by the router to forward the DHCP traffic to the Server.
On the router you need to configure this on the interface where the DHCP client is or where the Client Default gateway is located

![DHCP Relay](/static/images/DHCP-Relay.png)

```cisco
enable
confgiure terminal
interface gig0/2
ip 
```

### DHCP Client 

On the workstation PC we can try to contact the DHCP server and obtain the Address
and After executing the command we can see that the request goes through and i get an ip adddress 

![DHCP Client Test](/static/images/DHCP-Client-Work.png)

But i also tried on a NON configured Router Relay agent and of course it didnt work

![DHCP CLinet Failed](/static/images/DHCP-Client-Fail.png)

That fails because these types of messages that DHCP uses only work in LAN network, it uses Unicast and Broadcast, which cant work outside LAN (brodcast) 
I will now fix the HR device to obtain the address via DHCP from its approprate subnet and ping the 172.31.30.1 VPC

![DHCP Got Address](/static/images/DHCP-GotAddress.png)
![DHCP Ping works](/static/images/DHCP-PingWorks.png)

We can see that the HR device can contact the DEV device and see that it goes over the route via the Switch path this could be changed if we set up some priority but lets no complicate things.
Unfourtunately I don’t have access to ip dhcp snooping commands over my Vios router cos I think it is required to have a license for that. So you can try using something from the CSR Router series. 
That is it for the DHCP, we have seen how it works on Real Cisco gear and how to configure the Server outside its LAN network

