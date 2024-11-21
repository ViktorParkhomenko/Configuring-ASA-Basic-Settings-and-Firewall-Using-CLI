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




• Configure basic ASA settings and interface security levels using CLI

Configure the hostname and domain name.
Configure the ASA hostname as ASA.
Configure the enable mode password.
Use the enable password command to change the privileged EXEC mode password to cisco

![{45BF3702-E02B-47CC-B8DF-62E313A78717}](https://github.com/user-attachments/assets/25dc6bfe-2831-4785-82b2-38e4f335eac1)




• Configure routing, address translation, and inspection policy using CLI

• Configure DHCP, AAA, and SSH

• Configure a DMZ, Static NAT, and ACLs
