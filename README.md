# Comprehensive Cisco Packet Tracer Lab Guide

This guide breaks down the essential networking protocols needed to build a fully functional enterprise network simulation. It covers dynamic IP assignment, voice communications, file management, internet translation, and time synchronization.

---

## 1. DHCP (Dynamic Host Configuration Protocol)
DHCP automates the process of assigning IP addresses, subnet masks, default gateways, and other parameters to devices on a network. This prevents IP conflicts and saves you from manually configuring hundreds of PCs.

### **How It Works**
When a device connects to the network, it broadcasts a request. The router (acting as the DHCP server) responds with an available IP address from a predefined "pool."

### **Key Configuration Steps**
1. Exclude the gateway IPs so the router doesn't assign its own address.
2. Create the DHCP pool and define the network range.
3. Set the default router (gateway) and any specific options.

### **Example Configuration**
```
Router(config)# ip dhcp excluded-address 192.168.1.1
Router(config)# ip dhcp pool DATA_POOL
Router(dhcp-config)# network 192.168.1.0 255.255.255.0
Router(dhcp-config)# default-router 192.168.1.1
Router(dhcp-config)# dns-server 8.8.8.8
```

### **DHCP Relay (IP Helper)**
If your DHCP server is on a different router than your PCs or Phones, routers will block the DHCP broadcast. You must tell the local router interface to forward requests directly to the centralized DHCP server using `ip helper-address`.

```
Router(config)# interface FastEthernet0/0
Router(config-if)# ip helper-address 10.0.0.1
```

---

## 2. VoIP (Voice over IP) & CME
Voice over IP allows telephone calls to be routed over standard data networks. In Cisco Packet Tracer, routers act as a Call Manager Express (CME) to handle phone registrations and assign extensions.

### **How It Works**
IP Phones must first get an IP address and the TFTP server address (via DHCP Option 150) to download their configurations. They then register with the `telephony-service` and are assigned an extension number (`ephone-dn`). 

### **Example Configuration**

**Part A: The Voice DHCP Pool**
```
Router(config)# ip dhcp pool VOICE_POOL
Router(dhcp-config)# network 192.168.2.0 255.255.255.0
Router(dhcp-config)# default-router 192.168.2.1
Router(dhcp-config)# option 150 ip 192.168.2.1
```

**Part B: The Call Manager & Extensions**
```
Router(config)# telephony-service
Router(config-telephony)# max-ephones 5
Router(config-telephony)# max-dn 5
Router(config-telephony)# ip source-address 192.168.2.1 port 2000
Router(config-telephony)# auto assign 1 to 5
Router(config-telephony)# exit

Router(config)# ephone-dn 1
Router(config-ephone-dn)# number 1001
```

**Part C: Routing Calls to Another Router**
```
Router(config)# dial-peer voice 1 voip
Router(config-dial-peer)# destination-pattern 2...
Router(config-dial-peer)# session target ipv4:10.0.0.2
```

---

## 3. FTP (File Transfer Protocol)
FTP is used to transfer files between devices. In network administration, it is primarily used to back up router configurations (`running-config`) or upgrade the Cisco IOS software images to a centralized server.

### **Example Configuration**
```
! Optional: Set default credentials if your FTP server requires login
Router(config)# ip ftp username admin
Router(config)# ip ftp password cisco

! Backup the configuration to the server
Router# copy running-config ftp:
Address or name of remote host []? 192.168.1.100
Destination filename [router-confg]? backup-config.txt
```

---

## 4. NAT (Network Address Translation)
NAT translates private IP addresses (which cannot route on the public internet) into public IP addresses. When a packet leaves the internal network, NAT translates its local IP to a global IP, and reverses the process when packets return.

### **Terminology of NAT**
* **Inside Local:** A region inside the Enterprise network where hosts have Private IP addresses.
* **Inside Global:** A region inside the Enterprise network that uses Public IP addresses (usually connected to the outside network/Internet).
* **Outside Local:** A region in a public Internet where hosts have private IP addresses.
* **Outside Global:** A region in a public Internet where Public IP addresses are used.

### **Static NAT**
In Static NAT, IP addresses are statically mapped to each other through manual configuration. There are two types:

**1. Inside Static NAT**
This involves the static mapping of the Inside Local IP (private) to the Inside Global IP (public). Private IP addresses remain hidden from the outside network.

```
R1(config)# int f0/0
R1(config-if)# ip nat outside
R1(config-if)# exit

R1(config)# int f1/0
R1(config-if)# ip nat inside
R1(config-if)# exit

! Enable Inside Static NAT mapping
R1(config)# ip nat inside source static 10.1.1.2 20.1.1.1
```

**2. Outside Static NAT**
This involves the static mapping of the Outside Global IP (public) to an Outside Local IP (private). The real external IP addresses remain hidden from the hosts.

```
R1(config)# int f0/0
R1(config-if)# ip nat outside
R1(config-if)# exit

R1(config)# int f1/0
R1(config-if)# ip nat inside
R1(config-if)# exit

! Enable Outside Static NAT mapping
R1(config)# ip nat outside source static 30.1.1.1 192.168.1.2
```

---

## 5. NTP (Network Time Protocol)
NTP synchronizes the clocks of network devices to a central time server. This is critical for security, logging, and troubleshooting, as system logs are useless if every router has a different timestamp.

### **Example Configuration**
```
! Point the router to the NTP server
Router(config)# ntp server 192.168.1.50

! Verify the synchronization
Router# show ntp status
Router# show clock
```

---

## Summary Command Reference Table

| Protocol | Primary Purpose | Key Verification Command |
| :--- | :--- | :--- |
| **DHCP** | Auto-assigns IP addresses | `show ip dhcp binding` |
| **VoIP** | Manages voice calls and extensions | `show ephone registered` |
| **FTP** | Backs up configs and IOS images | `dir flash:` |
| **NAT** | Translates private IPs to public IPs | `show ip nat translations` |
| **NTP** | Synchronizes device clocks | `show ntp status` |
