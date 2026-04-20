# 230953444_ndp

ip dhcp pool voice_pool 

network 192.168.2.0 255.255.25.0

default-router 192.168.2.1

option 150 ip 192.168.2.1



telephony-service

max-ephones 5

max-dn 5

ip source-address 192.168.2.1 port 2000

exit 



ephone-dn 1

number xxxx



dial-peer voice 1 voip

destination-pattern 2...

session target ipv4:10.0.0.2



interface range <name>

sitchport mode access

switchport voice vlan 1




When you configure OSPF and RIP in different parts of the network (like your diagram), they do NOT automatically understand each other’s routes. That’s where things change.

🔹 What stays the same

For each protocol, configuration is normal:

✅ OSPF side (yellow region)
router ospf 1
network 192.168.0.0 0.0.0.255 area 0
network 192.172.18.0 0.0.0.255 area 0
network 192.152.17.0 0.0.0.255 area 0
✅ RIP side (white region)
router rip
version 2
network 172.0.0.0
network 192.192.18.0
no auto-summary

So far → same as usual single-protocol configs

🔴 What’s DIFFERENT (important)
👉 You MUST configure route redistribution on the border router

(In your diagram → Router2)

Because:

OSPF knows only OSPF routes
RIP knows only RIP routes
Without redistribution → networks won’t communicate fully
🔹 Redistribution Config
On Router2:
🔁 Inject RIP routes into OSPF
router ospf 1
redistribute rip subnets
🔁 Inject OSPF routes into RIP
router rip
redistribute ospf 1 metric 2
⚠️ Why metric is needed in RIP?

RIP requires a metric (hop count), so:

metric 2

Otherwise routes may not propagate correctly.

🔹 Other small things to keep in mind
Use no auto-summary in RIP (very important in modern networks)
Ensure interfaces are in correct networks
Check routing table using:
show ip route

You should see:

O → OSPF routes
R → RIP routes
O E2 or similar → redistributed routes
🔹 Your Hello Timer part (extra question d)

For Router2:

interface <interface-name>
ip ospf hello-interval 20
🔥 Final takeaway

👉 If using one protocol → simple config
👉 If using multiple protocols (OSPF + RIP) →

You MUST add redistribution, otherwise the networks remain partially isolated



VoIP in Packet Tracer is not “automatic”—you have to configure a call manager (CME) + assign numbers (extensions) + make sure phones get IPs. It’s a few steps, but once you see the flow, it’s straightforward.

🔹 What you’re basically doing

You need:

IP Phones get IP (via DHCP)
Router acts as Call Manager (CME)
Assign extensions (phone numbers)
Phones register → then you can call
🧠 In YOUR topology
IP Phone0 → Router2 (OSPF side)
IP Phone1 → Router1 (RIP side)

👉 You can configure VoIP on both routers separately
(Each router = mini call manager)

🔹 Step 1: Enable DHCP for phones

Phones need IP automatically.

On Router (example Router2)
ip dhcp pool VOICE
network 192.152.17.0 255.255.255.0
default-router 192.152.17.1
option 150 ip 192.152.17.1

👉 option 150 tells phones where the call manager is

🔹 Step 2: Enable Call Manager Express (CME)
telephony-service
max-ephones 5
max-dn 5
ip source-address 192.152.17.1 port 2000
auto assign 1 to 2
🔹 Step 3: Create extensions (phone numbers)
ephone-dn 1
number 1001

ephone-dn 2
number 1002
🔹 Step 4: Register phones

When phones connect:

They get IP from DHCP
They auto-register to router

You can check:

show ephone registered
🔹 Step 5: Repeat on other router

On Router1 (RIP side), do same but different network:

Example:

ip dhcp pool VOICE
network 192.192.18.0 255.255.255.0
default-router 192.192.18.1
option 150 ip 192.192.18.1

Then:

telephony-service
max-ephones 5
max-dn 5
ip source-address 192.192.18.1 port 2000

ephone-dn 1
number 2001
🔴 IMPORTANT (for calling between routers)

Right now:

Router2 phones → 1001, 1002
Router1 phones → 2001

👉 These are different networks, so you must route calls between routers.

🔹 Step 6: Configure Dial Peers (VERY IMPORTANT)
On Router2:
dial-peer voice 1 voip
destination-pattern 2...
session target ipv4:172.0.0.2
On Router1:
dial-peer voice 1 voip
destination-pattern 1...
session target ipv4:172.0.0.1

👉 Replace IPs with actual router-to-router IPs

🔹 Step 7: Test call

From IP Phone0:
➡️ Dial 2001
→ Should ring IP Phone1

⚠️ Common mistakes (you’ll lose marks here)
❌ Forgot option 150 → phones won’t register
❌ No dial-peer → cross-router call fails
❌ Wrong IP in session target
❌ Phones not in correct VLAN/network
🔥 Final idea (very important for exam)

👉 VoIP = DHCP + CME + Extensions + Dial Peer

If same router → no dial peer needed
If different routers → dial peer is compulsory
