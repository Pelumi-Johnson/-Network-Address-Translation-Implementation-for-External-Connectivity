# Network Address Translation Implementation for External Connectivity

## Overview
Designed and implemented Network Address Translation to enable a private internal network to communicate with an external network. Configured dynamic NAT overload to translate private IP addresses into a routable public IP, simulating real world internet access.

## Objective
Established external connectivity for a private network by implementing NAT, allowing internal hosts to communicate with external networks through address translation.

## Network Setup
Devices Used
1 Cisco 2960 Switch
2 Cisco 1941 Routers
1 End Device PC

Topology Design
PC connected to switch inside network
Switch connected to primary router internal interface
Primary router connected to ISP router via external interface
ISP router simulates internet network

Logical Flow
PC to Switch to Internal Router to ISP Router

Topology
![Network Topology](https://github.com/Pelumi-Johnson/-Network-Address-Translation-Implementation-for-External-Connectivity/blob/main/Screenshot%202026-04-15%20222826.png)

## Configuration

### Inside Network Configuration

Configured internal interface on router and assigned private IP range

interface gigabitEthernet 0/0
ip address 192.168.10.1 255.255.255.0
no shutdown

Configured end device with private addressing

IP 192.168.10.10
Subnet Mask 255.255.255.0
Default Gateway 192.168.10.1

Inside Configuration
![Inside Config](https://github.com/Pelumi-Johnson/-Network-Address-Translation-Implementation-for-External-Connectivity/blob/main/Screenshot%202026-04-15%20223154.png)

---

![Inside Config](https://github.com/Pelumi-Johnson/-Network-Address-Translation-Implementation-for-External-Connectivity/blob/main/Screenshot%202026-04-15%20223021.png)


### Outside Network Configuration

Configured external interface on primary router with public IP

interface gigabitEthernet 0/1
ip address 200.1.1.1 255.255.255.0
no shutdown

Configured ISP router interface

interface gigabitEthernet 0/0
ip address 200.1.1.2 255.255.255.0
no shutdown

Outside Configuration
![Outside Config](https://github.com/Pelumi-Johnson/-Network-Address-Translation-Implementation-for-External-Connectivity/blob/main/Screenshot%202026-04-15%20223610.png)

### NAT Configuration

Defined inside and outside interfaces

interface gigabitEthernet 0/0
ip nat inside

interface gigabitEthernet 0/1
ip nat outside

Configured NAT rule using overload for dynamic translation

access-list 1 permit 192.168.10.0 0.0.0.255
ip nat inside source list 1 interface gigabitEthernet 0/1 overload

NAT Configuration
![NAT Config](https://github.com/Pelumi-Johnson/-Network-Address-Translation-Implementation-for-External-Connectivity/blob/main/Screenshot%202026-04-15%20223706.png)

---

![NAT Config](https://github.com/Pelumi-Johnson/-Network-Address-Translation-Implementation-for-External-Connectivity/blob/main/Screenshot%202026-04-15%20224128.png)
## Validation

### Connectivity Testing

Tested communication from internal host to external network

ping 200.1.1.2

Result successful confirming NAT translation and external reachability

Successful Ping
![Ping Success](https://github.com/Pelumi-Johnson/-Network-Address-Translation-Implementation-for-External-Connectivity/blob/main/Screenshot%202026-04-15%20224224.png)

---

![Tracert Success](https://github.com/Pelumi-Johnson/-Network-Address-Translation-Implementation-for-External-Connectivity/blob/main/Screenshot%202026-04-15%20224351.png)

### NAT Translation Verification

Verified active translations on router

show ip nat translations

Confirmed private IP was translated to public interface address

NAT Table
![NAT Table](https://github.com/Pelumi-Johnson/-Network-Address-Translation-Implementation-for-External-Connectivity/blob/main/Screenshot%202026-04-16%20003326.png)

## Failure Testing

Removed NAT configuration to simulate loss of translation

no ip nat inside source list 1 interface gigabitEthernet 0/1 overload

Retested connectivity

ping 200.1.1.2

Result failed confirming inability of private IP to reach external network without translation

NAT Failure
![NAT Failure](https://github.com/Pelumi-Johnson/-Network-Address-Translation-Implementation-for-External-Connectivity/blob/main/Screenshot%202026-04-16%20002343.png)

---

![Ping Failure](https://github.com/Pelumi-Johnson/-Network-Address-Translation-Implementation-for-External-Connectivity/blob/main/Screenshot%202026-04-16%20002514.png)

---

### Step 1 — Attempted Fix (Still Failed)

Reapplied NAT rule but connectivity still did not work

ping 200.1.1.2

Result failed indicating deeper issue beyond missing NAT rule

Retry Failure
![Retry Failure](https://github.com/Pelumi-Johnson/-Network-Address-Translation-Implementation-for-External-Connectivity/blob/main/Screenshot%202026-04-16%20012330.png)

---

![Retry Failure](https://github.com/Pelumi-Johnson/-Network-Address-Translation-Implementation-for-External-Connectivity/blob/main/Screenshot%202026-04-16%20010145.png)

---

### Step 2 — Path Analysis (Tracing the Failure)

Executed tracert to identify where communication breaks

tracert 200.1.1.2

Observation:
Packets successfully reached default gateway (192.168.10.1)
Failure occurred beyond the gateway

Conclusion:
Issue is not Layer 2 or local subnet — problem exists at router translation or outbound path

Tracert Analysis
![Tracert Failure](https://github.com/Pelumi-Johnson/-Network-Address-Translation-Implementation-for-External-Connectivity/blob/main/Screenshot%202026-04-16%20010246.png)

---

### Step 3 — Interface Verification

Checked interface status

show ip interface brief

Observation:
All interfaces are up and operational

Conclusion:
Physical and Layer 3 interface states are not the issue

Interface Check
![Interface Status](https://github.com/Pelumi-Johnson/-Network-Address-Translation-Implementation-for-External-Connectivity/blob/main/Screenshot%202026-04-16%20010318.png)

---

### Step 4 — Configuration Inspection (Root Cause Discovery)

Reviewed full configuration

show running-config

Observation:
- g0/0 correctly configured with ip nat inside
- g0/1 missing ip nat outside

Root Cause Identified:
NAT requires both inside and outside interfaces
Missing ip nat outside prevents translation entirely

Config Review
![Running Config](https://github.com/Pelumi-Johnson/-Network-Address-Translation-Implementation-for-External-Connectivity/blob/main/Screenshot%202026-04-16%20010341.png)

---

### Step 5 — Fix Applied

Configured outside interface correctly

interface gigabitEthernet 0/1
ip nat outside

Reapplied NAT rule

ip nat inside source list 1 interface gigabitEthernet 0/1 overload

NAT Fix Config
![NAT Fix Config](https://github.com/Pelumi-Johnson/-Network-Address-Translation-Implementation-for-External-Connectivity/blob/main/Screenshot%202026-04-16%20010358.png)

---

![NAT Fix Config](https://github.com/Pelumi-Johnson/-Network-Address-Translation-Implementation-for-External-Connectivity/blob/main/Screenshot%202026-04-16%20010414.png)

---

### Step 6 — Final Validation

Retested connectivity

ping 200.1.1.2

Result successful confirming NAT translation restored

NAT Success
![NAT Success](https://github.com/Pelumi-Johnson/-Network-Address-Translation-Implementation-for-External-Connectivity/blob/main/Screenshot%202026-04-16%20010436.png)

---


## Key Concepts Applied
- Network Address Translation for private to public communication
- Dynamic NAT overload for multiple host translation using single public IP
- Inside and outside interface designation
- ACL usage to define translatable traffic
- End to end connectivity validation using ICMP
- Verification of translation tables for operational visibility
- NAT dependency on correct inside and outside interface designation with dynamic overload (PAT) for multi host translation
- Use of tracert and verification commands (show ip interface brief and show running-config) to isolate failures beyond the gateway
- End to end troubleshooting by following packet flow from private network through translation to external destination

## Outcome
Enabled external network communication for a private subnet through NAT implementation, allowing internal hosts to access external resources using a single public IP. Validated translation behavior and confirmed dependency on NAT for successful internet communication, reflecting real world enterprise network design.
