# Comprehensive Cisco Packet Tracer Lab Guide

This guide breaks down the essential networking protocols needed to build a fully functional enterprise network simulation. It covers dynamic IP assignment, voice communications, file management, internet translation, and time synchronization.

---

## 1. DHCP (Dynamic Host Configuration Protocol)
DHCP automates the process of assigning IP addresses, subnet masks, default gateways, and other parameters to devices on a network. This prevents IP conflicts and saves you from manually configuring hundreds of PCs.

### **How It Works**
When a device connects to the network, it broadcasts a request. The router (acting as the DHCP server) responds with an available IP address from a predefined "pool."

### **Key Configuration Steps**
1. Exclude the gateway IPs so the router doesn't assign its own address to a PC.
2. Create the DHCP pool and define the network range.
3. Set the default router (gateway) and any specific options (like DNS or Option 150 for VoIP).

### **Example Configuration**
```
Router(config)# ip dhcp excluded-address 192.168.1.1
Router(config)# ip dhcp pool DATA_POOL
Router(dhcp-config)# network 192.168.1.0 255.255.255.0
Router(dhcp-config)# default-router 192.168.1.1
Router(dhcp-config)# dns-server 8.8.8.8
```

---

## 2. VoIP (Voice over IP) & CME
Voice over IP allows telephone calls to be routed over standard data networks. In Cisco Packet Tracer, routers act as a Call Manager Express (CME) to handle phone registrations and assign extensions.

### **How It Works**
IP Phones get their IP address and TFTP server address (via DHCP Option 150). They download their configuration from the router, register with the `telephony-service`, and are assigned an extension number (`ephone-dn`). If calling a different router, a `dial-peer` is required to route the traffic.

### **Example Configuration**
**Part A: The Call Manager**
` ``text
Router(config)# telephony-service
Router(config-telephony)# max-ephones 5
Router(config-telephony)# max-dn 5
Router(config-telephony)# ip source-address 192.168.2.1 port 2000
Router(config-telephony)# auto assign 1 to 5
` ``

**Part B: Assigning Extensions**
` ``text
Router(config)# ephone-dn 1
Router(config-ephone-dn)# number 1001
` ``

**Part C: Routing Calls to Another Router**
` ``text
Router(config)# dial-peer voice 1 voip
Router(config-dial-peer)# destination-pattern 2...
Router(config-dial-peer)# session target ipv4:10.0.0.2
` ``

---

## 3. FTP (File Transfer Protocol)
FTP is used to transfer files between devices. In network administration, it is primarily used to back up router configurations (`running-config`) or upgrade the Cisco IOS software images to a centralized server.

### **How It Works**
FTP operates on a client-server model using TCP ports 20 and 21. You configure an FTP server in your topology, ensure the router can ping it, and then execute the copy commands.

### **Example Configuration**
` ``text
! Optional: Set default credentials if your FTP server requires login
Router(config)# ip ftp username admin
Router(config)# ip ftp password cisco

! Backup the configuration to the server
Router# copy running-config ftp:
Address or name of remote host []? 192.168.1.100
Destination filename [router-confg]? backup-config.txt
` ``

---

## 4. NAT (Network Address Translation)
NAT translates private IP addresses (which cannot route on the public internet) into a public IP address. The most common form in Packet Tracer is **PAT (Port Address Translation)** or "NAT Overload," which allows an entire local network to share a single public WAN IP.

### **How It Works**
The router identifies internal traffic (`inside`) trying to reach the internet (`outside`). It checks an Access Control List (ACL) to see if the traffic is permitted, translates the private IP to the public IP, and tracks the session using port numbers.

### **Example Configuration**
` ``text
! 1. Define inside and outside interfaces
Router(config)# interface FastEthernet0/0
Router(config-if)# ip nat inside
Router(config)# interface FastEthernet0/1
Router(config-if)# ip nat outside

! 2. Create an ACL for your local network
Router(config)# access-list 1 permit 192.168.2.0 0.0.0.255

! 3. Apply the Overload rule
Router(config)# ip nat inside source list 1 interface FastEthernet0/1 overload
` ``

---

## 5. NTP (Network Time Protocol)
NTP synchronizes the clocks of network devices to a central time server. This is critical for security, logging, and troubleshooting, as system logs are useless if every router has a different timestamp.

### **How It Works**
You designate one device (usually a dedicated server in Packet Tracer) as the NTP master. All routers are then configured to point to that server's IP address to sync their internal hardware clocks.

### **Example Configuration**
` ``text
! Point the router to the NTP server
Router(config)# ntp server 192.168.1.50

! Verify the synchronization
Router# show ntp status
Router# show clock
` ``

---

## Summary Command Reference Table

| Protocol | Primary Purpose | Key Verification Command |
| :--- | :--- | :--- |
| **DHCP** | Auto-assigns IP addresses | `show ip dhcp binding` |
| **VoIP** | Manages voice calls and extensions | `show ephone registered` |
| **FTP** | Backs up configs and IOS images | `dir flash:` |
| **NAT** | Translates private IPs to public IPs | `show ip nat translations` |
| **NTP** | Synchronizes device clocks | `show ntp status` |
