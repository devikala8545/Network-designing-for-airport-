# Network-designing-for-airport-
IMPLEMETATION:
i)The DHCP server is connected to port, which is a member of VLAN 2 The IP address of DHCP server of Flight server authority is 192.168.3.2 and IP address of airport authority server is 192.168.2.2
ii)Access points are configured with IP address belonging to the VLAN 4network address range.
iii)Switch Configuration
Detailed configuration details on the switches in cicso switch is required. a. Create the VLAN’s lines namely VLAN 2, VLAN 3, and VLAN 4 with respect to the switch
switch(config)#vlan 2
switch(config-vlan)#name Airport authority switch(config-vlan)#exit
switch(config)#vlan 3
switch(config-vlan)#name Flight service providers switch(config-vlan)#exit
switch(config)#vlan 4 switch(config-vlan)#name Guests
  
switch(config-vlan)#exit
b. Now let’s configure appropriate ports on the switch as members of respective VLAN. Only two ports for each vlans are displayed and that can be added based on requirement.
switch(config)#interface fa 0/2 switch(config-if)#switchport mode access switch(config-if)#switchport access vlan 2
switch(config)#interface fa 0/3 switch(config-if)#switchport mode access switch(config-if)#switchport access vlan 2
switch(config)#interface fa 0/4 switch(config-if)#switchport mode access
switch(config-if)#switchport access vlan 2
and config the other Fa interface to VLAN 2 as shown above from switch 0 Similarly for Switch 1 and 2 configuration is made with Vlan 3 and 4 respectively
For Switch 1
switch(config)#interface fa 0/2 switch(config-if)#switchport mode access switch(config-if)#switchport access vlan 3
For all the Fa interface to VLAN 3 as shown above. For Switch 2
switch(config)#interface fa 0/1 switch(config-if)#switchport mode access switch(config-if)#switchport access vlan 4
For all the Fa interface to VLAN 4 as shown above.
c. Configure the port connected to the router as trunk. This enables in allowing traffic from all the vlans to the router where appropriate routing and access restriction are performed.
switch(config)#interface fa 0/1 switch(config-if)#switchport mode trunk switch(config-if)#switchport trunk allowed vlan all

switch(config-if)#exit
iv)Router configuration
The below configuration is for the router in the Packet diagram
a. The interface connected to the internet is configured with the appropriate
IP address.
router(config)#interface fa 0/0 router(config-if)#ip address 50.1.1.2 255.0.0.0
router(config-if)#no shutdown router(config-if)#exit router(config)#interface fa 0/1 router(config-if)#ip address 8.8.8.1 255.0.0.0 router(config-if)#no shutdown
b. Sub interfaces on the router on physical interface Fa0/0 are mapped with appropriate VLAN and IP address. These configured address on router
are default gateway address for users for respective VLAN.
router(config)#interface fa 0/0.1
router (conf ig-subif)#encapsulation dot1Q 2 router(config-subif)#ip address 192.168.2.1 255.255.255.0 router(config-subif)#no shutdown router(config-subif)#exit
router(config)#interface fa 0/0.2 router(config-subif)#encapsulation dot1Q 3 router(config-subif)#ip address 192.168.3.1 255.255.255.0 router(config-subif)#no shutdown router(config-subif)#exit
router(config)#interface fa 0/0.3 router(config-subif)#encapsulation dot1Q 4 router(config-subif)#ip address 192.168.4.1 255.255.255.0 router(config-subif)#no shutdown router(config-subif)#exit
c. The IP helper address is configured on VLAN 3 and 4 interface of router. This is configured for uses in their respective VLANs to reach DHCP
 
server for obtaining dynamic IP address. The configuration is
router(config)#interface fa 0/0.1 router(config-subif)#ip helper-address 192.168.2.2 router(config-subif)#exit
router(config)#interface fa 0/0.7
router(config-subif)#ip helper-address 192.168.3.2 router(config-subif)#exit
router(config)#interface fa 0/0.3 router(config-subif)#ip helper-address 192.168.4.2 router(config-subif)#exit
d. Appropriate access control list is configured on router. This is to deny access from guest network to other 2 networks which are an extended ACL. The first 2 lines deny access from guest network to airport authority and Flight server provider networks. Third entry allows all other traffic. This is for internet connection and the access control list is applied in guest vlan interface on router as inbound.
router(config)#access-list 101 deny ip 192.168.4.0 0.0.0.255
192.168.2.0 0.0.0.255
router(config)#access-list 101 deny ip 192.168.4.0 0.0.0.255
192.168.3.0 0.0.0.255
router(config)#access-list 101 permit ip any any
router(config)#interface fa 0/0.2 router(config-subif)#ip access-group 101 inbound
e. Access control list to restrict access from flight service network to airport authority network. The first line allows flight service provider network to access the airport authority. The second line denies all communication to airport authority network and third line allows all other communications that is for internet. Access list is applied inbound on VLAN interface corresponding to airport authority network.
router(config)#access-list 102 permit ip 192.168.2.0 0.0.0.255 host 192.168.3.2
router(config)#access-list 102 deny ip 192.168.3.0 0.0.0.255

192.168.2.0 0.0.0.255
router(config)#access-list 101 permit ip any any router(config)#interface fa 0/0.7
router(config-subif)#ip access-group 102 inbound
v) Firewall Configuration
We used ASA1 firewall in this design as it can work as a bridge between Vlan’s when configured. Considering the restrictions of access between the Vlans this is best way to config and implement the design
ciscoasa(config)#interface vlan 2 ciscoasa(config)#nameif inside ciscoasa(config-if)#security-level 100 ciscoasa(config-if)#ip address 192.168.2.0 255.255.255.0 ciscoasa(config-if)#exit
ciscoasa(config)#interface vlan 2 ciscoasa(config)#nameif outside ciscoasa(config-if)#security-level 0 ciscoasa(config-if)#ip address 192.168.2.0 255.255.255.0 ciscoasa(config-if)#exit
Similarity we restrict the IP address such that no guest can access Flight service provider or Airport authority and Flight service can’t access Airport authority only where as Airport authority has access to all the Vlans.
vi) DHCP Configuration
DHCP configuration are made to assign IP automatically to the end devices. For this process we gave a pool of address encapsulated such that an IP address is assigned to end devices automatically.
Router#sh ip dhcp pool
Router#conf t
Enter configuration commands, one per line. End with CNTL/Z. Router(config)#ip dhcp pool dv2
Router(dhcp-config)#network 192.168.2.0 255.255.255.0 Router(dhcp-config)#default-router 192.168.2.1 Router(dhcp-config)#%DHCPD-4-PING_CONFLICT: DHCP address conflict: server pinged 192.168.2.1.
  
Router(dhcp-config)#ip dhcp pool dv3 Router(dhcp-config)#network 192.168.3.0 255.255.255.0 Router(dhcp-config)#default-router 192.168.3.1 Router(dhcp-config)#ip dhcp pool dv4 Router(dhcp-config)#network 192.168.4.0 255.255.255.0 Router(dhcp-config)#network 192.168.4.0 255.255.255.0 %DHCPD-4-PING_CONFLICT: DHCP address confl Router(dhcp-config)#default-router 192.168.4.1 Router(dhcp-config)#ex
Router(config)#ex
Router#%SYS-5-CONFIG_I: Configured from console by console Router#sh run
Building configuration...
Current configuration : 989 bytes
!
version 12.2
no service timestamps log datetime msec
no service timestamps debug datetime msec no service password-encryption
!
hostname Router
! !
ip dhcp pool dv2
network 192.168.2.0 255.255.255.0 default-router 192.168.2.1
ip dhcp pool dv3
network 192.168.3.0 255.255.255.0 default-router 192.168.3.1
ip dhcp pool dv4
network 192.168.4.0 255.255.255.0 default-router 192.168.4.1
