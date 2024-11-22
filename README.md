# Configuring ASA Basic Settings and Firewall Using CLI

Project Overview
This project simulates the configuration of an ASA (Adaptive Security Appliance) in my home lab using Cisco Packet Tracer. The ASA device connects the internal network (Inside), the DMZ, and the ISP (Outside), and provides services such as NAT, DHCP, and Firewall policies.

ASA acts as the firewall, security device, and NAT provider, with interfaces configured for three zones: Inside, Outside, and DMZ.
Inside network: A private network (192.168.1.0/24) connected to the ASA's Inside interface.
Outside network: A public IP subnet (209.165.200.224/29) connected to the ASA's Outside interface.
DMZ network: A network used for a public-facing web server (192.168.2.0/24) connected to the ASA's DMZ interface.
This setup allows me to practice Basic ASA Configuration, Routing, NAT (PAT), ACLs, SSH, and AAA Authentication.

![{424BA683-3CA8-42A8-9DDC-D5A9445337FA}](https://github.com/user-attachments/assets/f50c6396-61dd-47a1-84ef-e8c93470c3a3)

Part 1: Verify Connectivity and Explore the ASA
Step 1: Test ASA Interface Connectivity
Ping the Inside Interface (192.168.1.1) from PC-B to verify connectivity. This should be successful.
Ping the Outside Interface (209.165.200.226) from PC-B to confirm it's inaccessible (which should fail due to the default security policy).

Part 2: Configure ASA Basic Settings and Interface Security Using CLI
Step 1: Set ASA Hostname and Domain Name

\r ASA(config)# hostname ASA
\r ASA(config)# domain-name ciscosecurity.com

Step 2: Configure Enable Password
Set the enable password to ciscoenpa55 for privileged EXEC mode access.

ASA(config)# enable password ciscoenpa55

Step 3: Configure ASA Interfaces
Inside Interface Configuration
The inside interface is configured for the private network (192.168.1.0/24):

ASA(config)# interface vlan 1
ASA(config-if)# nameif inside
ASA(config-if)# ip address 192.168.1.1 255.255.255.0
ASA(config-if)# security-level 100

Outside Interface Configuration
The outside interface is configured for the public network (209.165.200.224/29):

ASA(config)# interface vlan 2
ASA(config-if)# nameif outside
ASA(config-if)# ip address 209.165.200.226 255.255.255.248
ASA(config-if)# security-level 0

![{45BF3702-E02B-47CC-B8DF-62E313A78717}](https://github.com/user-attachments/assets/25dc6bfe-2831-4785-82b2-38e4f335eac1)

Step 4: Verify Interface Configurations
Test connectivity by:

Pinging the Inside interface (192.168.1.1) from PC-B (ping should succeed).

Attempting to ping the Outside interface (209.165.200.226) from PC-B (ping should fail).

![{3F12DBB2-DEA4-4337-A05C-FB98320E1D66}](https://github.com/user-attachments/assets/b04facfe-3395-4078-9718-fe3099fe6f37)

Part 3: Configure Routing, Address Translation, and Inspection Policy
Step 1: Configure a Static Default Route
Create a default route to allow the ASA to reach external networks:

ASA(config)# route outside 0.0.0.0 0.0.0.0 209.165.200.225

Verify the static route by checking the routing table:

ASA(config)# show route

Ping the R1 G0/0 interface (10.1.1.1) to ensure external connectivity:

![{9335420C-5822-4CC9-8F92-7DE4A7AD4DA2}](https://github.com/user-attachments/assets/408994ca-d9aa-4422-9086-e2921e98f740)

Step 2: Configure PAT and Network Objects
Configure PAT (Port Address Translation) for the Inside network, so internal devices can share the ASA's outside IP address:

ASA(config)# object network inside-net
ASA(config-network-object)# subnet 192.168.1.0 255.255.255.0
ASA(config-network-object)# nat (inside,outside) dynamic interface
ASA(config-network-object)# end

Verify the NAT translation hits by running:

ASA(config)# show nat

![{A93DCFF0-C02E-4095-86DF-14DDFD3B5F64}](https://github.com/user-attachments/assets/9290ad6c-160e-4f77-aa6d-13aadd442b9a)

Try to ping R1 G0/0 IP (209.165.200.225) from PC-B to observe that the pings fail due to the default security policy.

Step 3: Modify MPF (Modular Policy Framework) to Allow ICMP
To allow ICMP (ping) replies, modify the ASA’s MPF policy to inspect ICMP traffic:

ASA(config)# class-map inspection_default
ASA(config-cmap)# match default-inspection-traffic
ASA(config-cmap)# exit
ASA(config)# policy-map global_policy
ASA(config-pmap)# class inspection_default
ASA(config-pmap-c)# inspect icmp
ASA(config-pmap-c)# exit
ASA(config)# service-policy global_policy global

Test by pinging R1 G0/0 IP again. This time, the pings should succeed.

![{A97F0F5C-06F4-4B5C-BD3B-7BB7DB397AB6}](https://github.com/user-attachments/assets/64e3d0e8-6b5b-4a45-9d6d-61a57ebaa9ad)

Part 4: Configure DHCP, AAA, and SSH
Step 1: Configure DHCP on the ASA
Set up a DHCP server for the Inside network:

ASA(config)# dhcpd address 192.168.1.5-192.168.1.36 inside
ASA(config)# dhcpd dns 8.8.8.8 interface inside
ASA(config)# dhcpd enable inside

Step 2: Configure AAA Authentication Using the Local Database
Set up AAA (Authentication, Authorization, and Accounting) for SSH login using the local database:

ASA(config)# username admin password adminpa55
ASA(config)# aaa authentication ssh console LOCAL

Step 3: Configure SSH Access to ASA
Generate an RSA key pair for SSH connections:

ASA(config)# crypto key generate rsa modulus 1024

Allow SSH access from the Inside network (192.168.1.0/24) and a remote management host (172.16.3.3) on the Outside network:

ASA(config)# ssh 192.168.1.0 255.255.255.0 inside
ASA(config)# ssh 172.16.3.3 255.255.255.255 outside
ASA(config)# ssh timeout 10

Test SSH access from PC-C to the ASA’s Outside IP (209.165.200.226) and PC-B to the Inside IP (192.168.1.1).

![{CB3F47B6-454F-4FCB-A84E-D0631BEBBAF5}](https://github.com/user-attachments/assets/072441d9-75ed-4113-b20e-bc48531842c9)

![{28339A80-9C8D-4E71-A14E-053B3C1341BD}](https://github.com/user-attachments/assets/8e0b7c59-963d-4d4d-a44b-1f57917bee60)


Part 5: Configure a DMZ, Static NAT, and ACLs

Step 1: Configure the DMZ interface VLAN 3 on the ASA.

a. Configure DMZ VLAN 3, which is where the public access web server will reside. Assign it IP address
192.168.2.1/24, name it dmz, and assign it a security level of 70. Because the server does not need to
initiate communication with the inside users, disable forwarding to interface VLAN 1.

ASA(config)# interface vlan 3
ASA(config-if)# ip address 192.168.2.1 255.255.255.0
ASA(config-if)# no forward interface vlan 1
ASA(config-if)# nameif dmz
ASA(config-if)# security-level 70


b. Assign ASA physical interface E0/2 to DMZ VLAN 3 and enable the interface.

ASA(config)# interface Ethernet0/2
ASA(config-if)# switchport access vlan 3

![{A997FD2F-34AA-494D-A590-CBC3EC78F37B}](https://github.com/user-attachments/assets/974d96c2-5a7e-40e7-a18d-ff6747fb156c)

Step 2: Configure static NAT to the DMZ server using a network object.
Configure a network object named dmz-server and assign it the static IP address of the DMZ server
(192.168.2.3). While in object definition mode, use the nat command to specify that this object is used to
translate a DMZ address to an outside address using static NAT, and specify a public translated address of
209.165.200.227.

ASA(config)# object network dmz-server
ASA(config-network-object)# host 192.168.2.3
ASA(config-network-object)# nat (dmz,outside) static 209.165.200.227
ASA(config-network-object)# exit

Step 3: Configure an ACL to allow access to the DMZ server from the Internet.
Configure a named access list OUTSIDE-DMZ that permits the TCP protocol on port 80 from any external
host to the internal IP address of the DMZ server. Apply the access list to the ASA outside interface in the “IN”
direction.

ASA(config)# access-list OUTSIDE-DMZ permit icmp any host 192.168.2.3
ASA(config)# access-list OUTSIDE-DMZ permit tcp any host 192.168.2.3 eq 80
ASA(config)# access-group OUTSIDE-DMZ in interface outside

Note: Unlike IOS ACLs, the ASA ACL permit statement must permit access to the internal private DMZ
address. External hosts access the server using its public static NAT address, the ASA translates it to the
internal host IP address, and then applies the ACL.

Step 4: Test access to the DMZ server.
Test outside access to the DMZ - launch a ping test to the DMZ server to check for connectivity. Then, try and
get to the web server from PC-C.

![{7F8E03DB-3993-4C2B-991C-40F0FF5AA59C}](https://github.com/user-attachments/assets/4776a7ad-c9e8-4dbd-b823-2cc32a475283)

![{DFA4FE8E-3B57-469F-BFC7-E9757BF4A6F3}](https://github.com/user-attachments/assets/ddf568f3-6230-416c-8796-77bcd7abd44a)
