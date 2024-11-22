# Configuring ASA Basic Settings and Firewall Using CLI

Scenario
Your company has one location connected to an ISP. R1 represents Customer Premises Equipment (CPE)
device managed by the ISP. R2 represents an intermediate Internet router. R3 represents an ISP that
connects an administrator from a network management company, who has been hired to remotely manage
your network. The ASA is an edge CPE security device that connects the internal corporate network and DMZ
to the ISP while providing NAT and DHCP services to inside hosts. The ASA will be configured for
management by an administrator on the internal network and by the remote administrator. Layer 3 VLAN
interfaces provide access to the three areas created in the activity: Inside, Outside, and DMZ. The ISP
assigned the public IP address space of 209.165.200.224/29, which will be used for address translation on
the ASA.


• Verify connectivity and explore the ASA

![{424BA683-3CA8-42A8-9DDC-D5A9445337FA}](https://github.com/user-attachments/assets/f50c6396-61dd-47a1-84ef-e8c93470c3a3)


Part 2: Configure ASA Settings and Interface Security Using the CLI

Configure the hostname and domain name.
Configure the ASA hostname as ASA.
Configure the enable mode password.
Use the enable password command to change the privileged EXEC mode password to cisco

Configure the inside and outside interfaces.

a. Configure a logical VLAN 1 interface for the inside network (192.168.1.0/24) and set the security level to
the highest setting of 100.
ASA(config)# interface vlan 1
ASA(config-if)# nameif inside
ASA(config-if)# ip address 192.168.1.1 255.255.255.0
ASA(config-if)# security-level 100
b. Create a logical VLAN 2 interface for the outside network (209.165.200.224/29), set the security level to
the lowest setting of 0, and enable the VLAN 2 interface.
ASA(config-if)# interface vlan 2
ASA(config-if)# nameif outside
ASA(config-if)# ip address 209.165.200.226 255.255.255.248
ASA(config-if)# security-level 0


![{45BF3702-E02B-47CC-B8DF-62E313A78717}](https://github.com/user-attachments/assets/25dc6bfe-2831-4785-82b2-38e4f335eac1)

Test connectivity to the ASA.
I should be able to ping from PC-B to the ASA inside interface address (192.168.1.1)
From PC-B, ping the VLAN 2 (outside) interface at IP address 209.165.200.226. You should not be able
to ping this address

![{3F12DBB2-DEA4-4337-A05C-FB98320E1D66}](https://github.com/user-attachments/assets/b04facfe-3395-4078-9718-fe3099fe6f37)

Part 3: Configure Routing, Address Translation, and Inspection Policy Using the CLI

Configure a static default route for the ASA.
Configure a default static route on the ASA outside interface to enable the ASA to reach external networks.
a. Create a “quad zero” default route using the route command, associate it with the ASA outside interface,
and point to the R1 G0/0 IP address (209.165.200.225) as the gateway of last resort.
ASA(config)# route outside 0.0.0.0 0.0.0.0 209.165.200.225
b. Issue the show route command to verify the static default route is in the ASA routing table.
Verify that the ASA can ping the R1 S0/0/0 IP address 10.1.1.1
![{9335420C-5822-4CC9-8F92-7DE4A7AD4DA2}](https://github.com/user-attachments/assets/408994ca-d9aa-4422-9086-e2921e98f740)

Configure address translation using PAT and network objects.
Create network object inside-net and assign attributes to it using the subnet and nat commands.

ASA(config)# object network inside-net
ASA(config-network-object)# subnet 192.168.1.0 255.255.255.0
ASA(config-network-object)# nat (inside,outside) dynamic interface
ASA(config-network-object)# end

The ASA splits the configuration into the object portion that defines the network to be translated and the
actual nat command parameters. These appear in two different places in the running configuration.

![{A93DCFF0-C02E-4095-86DF-14DDFD3B5F64}](https://github.com/user-attachments/assets/9290ad6c-160e-4f77-aa6d-13aadd442b9a)

From PC-B attempt to ping the R1 G0/0 interface at IP address 209.165.200.225. The pings should fail.

Issue the show nat command on the ASA to see the translated and untranslated hits. Notice that, of the
pings from PC-B, four were translated and four were not. The outgoing pings (echos) were translated and
sent to the destination. The returning echo replies were blocked by the firewall policy. You will configure
the default inspection policy to allow ICMP in Step 3 of this part of the activity.

 Modify the default MPF application inspection global service policy.
For application layer inspection and other advanced options, the Cisco MPF is available on ASAs.
The Packet Tracer ASA device does not have an MPF policy map in place by default. As a modification, we
can create the default policy map that will perform the inspection on inside-to-outside traffic. When configured
correctly only traffic initiated from the inside is allowed back in to the outside interface. You will need to add
ICMP to the inspection list.
a. Create the class-map, policy-map, and service-policy. Add the inspection of ICMP traffic to the policy map
list using the following commands:

ASA(config)# class-map inspection_default
ASA(config-cmap)# match default-inspection-traffic
ASA(config-cmap)# exit
ASA(config)# policy-map global_policy
ASA(config-pmap)# class inspection_default
ASA(config-pmap-c)# inspect icmp
ASA(config-pmap-c)# exit
ASA(config)# service-policy global_policy global

From PC-B, attempt to ping the R1 G0/0 interface at IP address 209.165.200.225. The pings should be
successful this time because ICMP traffic is now being inspected and legitimate return traffic is being
allowed. If the pings fail, troubleshoot your configurations.

![{A97F0F5C-06F4-4B5C-BD3B-7BB7DB397AB6}](https://github.com/user-attachments/assets/64e3d0e8-6b5b-4a45-9d6d-61a57ebaa9ad)

Part 4: Configure DHCP, AAA, and SSH
Configure the ASA as a DHCP server

ASA(config)# dhcpd address 192.168.1.5-192.168.1.36 inside
ASA(config)# dhcpd dns 8.8.8.8 interface inside
ASA(config)# dhcpd enable inside

Change PC-A and PC-B from a static IP address to a DHCP client, and verify that it receives IP addressing
information.
![{0F1B589B-FFA5-416A-9F0D-ED70BF20349D}](https://github.com/user-attachments/assets/df979fe7-4f18-448d-bb2c-416b7095e4fd)

Configure AAA to use the local database for authentication

ASA(config)# username admin password adminpa55
ASA(config)# aaa authentication ssh console LOCAL

Configure remote access to the ASA.

The ASA can be configured to accept connections from a single host or a range of hosts on the inside or
outside network. In this step, hosts from the outside network can only use SSH to communicate with the ASA.
SSH sessions can be used to access the ASA from the inside network.
a. Generate an RSA key pair, which is required to support SSH connections. Because the ASA device has
RSA keys already in place, enter no when prompted to replace them.

ASA(config)# crypto key generate rsa modulus 1024

b. Configure the ASA to allow SSH connections from any host on the inside network (192.168.1.0/24) and
from the remote management host at the branch office (172.16.3.3) on the outside network. Set the SSH
timeout to 10 minutes (the default is 5 minutes).

ASA(config)# ssh 192.168.1.0 255.255.255.0 inside
ASA(config)# ssh 172.16.3.3 255.255.255.255 outside
ASA(config)# ssh timeout 10

c. Establish an SSH session from PC-C to the ASA (209.165.200.226).
PC> ssh -l admin 209.165.200.226

![{CB3F47B6-454F-4FCB-A84E-D0631BEBBAF5}](https://github.com/user-attachments/assets/072441d9-75ed-4113-b20e-bc48531842c9)

d. Establish an SSH session from PC-B to the ASA (192.168.1.1). Troubleshoot if it is not successful.
PC> ssh -l admin 192.168.1.1

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



