# Cisco Networking Handbook 🚀
*A Practical Guide to Routing Protocols & Network Services*

## 📋 Table of Contents
- Project Overview
- Topology Description
- Protocol Guides
- EIGRP
- OSPF
- RIP
- NAT
- ACL
- DHCP



---

## 🎯 Project Overview

This repository contains a comprehensive Packet Tracer project demonstrating multiple networking protocols in an enterprise environment. The topology simulates a real-world network with multiple routing protocols, NAT, ACLs, and DHCP services.

**Key Features:**
- Multi-area OSPF deployment
- EIGRP and RIP integration
- NAT with static & dynamic configurations
- ACL implementation for security
- DHCP server configuration
- Inter-VLAN routing

---

## 🌐 Topology Description

### Network Architecture

<img width="1437" height="780" alt="Project Topology" src="https://github.com/user-attachments/assets/3e9b8184-bbbc-450c-b720-a79239aae06e" />

---

## 🔄 Protocol Guides

### 1. EIGRP (Enhanced Interior Gateway Routing Protocol)

**What it is:** Cisco-proprietary advanced distance vector protocol.

**Key Concepts:**
- Uses DUAL algorithm for loop-free paths
- Maintains topology table
- Supports unequal cost load balancing
- Fast convergence with feasible successors

**Configuration Template:**

```cisco
! Step 1: Enable EIGRP with AS number (1-65535)
router eigrp <AS_NUMBER>

! Step 2: Configure Router ID (optional but recommended, helpful while debugging)
 eigrp router-id <X.X.X.X>

! Step 3: Define which networks to advertise
 network <NETWORK_IP> <WILDCARD_MASK>

! Step 4: (Optional) Passive interfaces (don't send hellos)
 passive-interface <INTERFACE>

! Step 5: (Optional) Adjust metrics if needed
 metric weights 0 1 0 1 0 0   ! K-values: K1=1, K3=1, others=0

! Step 6: Summarize routes
 no auto-summary                 ! auto summary disrupted my routes most of the time
```

**Configuration Example:**

```cisco
router eigrp 100
 eigrp router-id 1.1.1.1
 network 10.1.0.0 0.0.255.255
 network 192.168.12.0 0.0.0.3
 no auto-summary
```

**Verification Commands:**

```cisco
show ip eigrp neighbors
show ip eigrp topology
show ip route eigrp
```

**Key Points:**
- AS Number must match on all routers in the same EIGRP domain
- Wildcard mask = inverse of subnet mask
- Router ID should be unique (usually loopback IP)

---

### 2. OSPF (Open Shortest Path First)

**What it is:** Link-state routing protocol using Dijkstra's algorithm.

**Key Concepts:**
- Hierarchical design with areas
- Uses cost as metric (10^8/BW)
- Establishes neighbor adjacencies
- Supports authentication

**Configuration Template:**

```cisco
router ospf <PROCESS_ID>
 router-id <X.X.X.X>
 network <NETWORK_IP> <WILDCARD_MASK> area <AREA_ID>
 passive-interface <INTERFACE>
```

**Configuration Example:**

```cisco
router ospf 1
 router-id 1.1.1.1
 network 10.1.0.0 0.0.255.255 area 0
 network 192.168.12.0 0.0.0.3 area 1
```

**Verification Commands:**

```bash
show ip ospf neighbor
show ip ospf database
show ip route ospf
```

**Key Points:**
- Use Area 0 for backbone
- Router ID should be unique
- Wildcard mask = inverse of subnet mask
- Process ID is locally significant
- Hello/dead timers must match on neighbors

---

### 3. RIP (Routing Information Protocol)

**What it is:** Simple distance vector protocol.

**Key Concepts:**
- Maximum hop count: 15
- Sends full routing table every 30 seconds
- Simple but slow convergence
- Version 2 supports VLSM

**Configuration Template:**

```cisco
router rip
 version 2
 network <NETWORK_CLASS>
 no auto-summary
```

**Configuration Example:**

```cisco
router rip
 version 2
 network 10.0.0.0
 network 192.168.1.0
 no auto-summary
```

**Verification Commands:**

```bash
show ip rip database
show ip route rip
debug ip rip
```

**Key Points:**
- Always use version 2 (classless support)
- Always use no auto-summary (prevents route issues)
- Network statement uses classful address
- Max hop count = 15
- Administrative Distance = 120

---

## NAT (Network Address Translation)

**What it is:** Translates private IPs to public IPs.

**Types:**
- **Static NAT:** One-to-one mapping
- **Dynamic NAT:** Many-to-many from pool
- **PAT (Overload):** Many-to-one with ports

**Configuration Template (Static NAT):**

```cisco
! Map single private to single public IP
ip nat inside source static <PRIVATE_IP> <PUBLIC_IP>

! Define interfaces
interface <INSIDE_INTERFACE>
 ip nat inside

interface <OUTSIDE_INTERFACE>
 ip nat outside
```

**Configuration Example (Static NAT):**

```cisco
! Static NAT
ip nat inside source static 192.168.1.10 203.0.113.5

! Interface configuration
interface GigabitEthernet0/0
 ip nat inside
interface GigabitEthernet0/1
 ip nat outside
```

**Dynamic NAT (Many-to-Many Pool) [not verified myself]:**

```cisco
! Define NAT pool
ip nat pool <POOL_NAME> <START_IP> <END_IP> netmask <SUBNET_MASK>

! Define ACL for internal hosts
access-list <ACL_NUMBER> permit <SOURCE_NETWORK> <WILDCARD>

! Apply NAT
ip nat inside source list <ACL_NUMBER> pool <POOL_NAME>

! Configure interfaces
interface <INSIDE_INTERFACE>
 ip nat inside
interface <OUTSIDE_INTERFACE>
 ip nat outside
```

**Verification Commands:**

```bash
show ip nat translations
show ip nat statistics
debug ip nat
```

---

## Standard ACL (Access Control Lists)

**What it is:** Filters traffic based on rules.

**Basic Configuration:**

```cisco
! Create ACL (numbers 1-99)
access-list <1-99> <permit|deny> <SOURCE> <WILDCARD>

! Apply to interface
interface <INTERFACE>
 ip access-group <ACL_NUMBER> <in|out>
```

**Minimal Examples:**

```cisco
! Block single host
access-list 10 deny host 192.168.1.100
access-list 10 permit any

! Block entire network
access-list 20 deny 192.168.2.0 0.0.0.255
access-list 20 permit any

! Allow only specific network
access-list 30 permit 10.1.1.0 0.0.0.255
access-list 30 deny any

! Apply to interface
interface FastEthernet0/0
 ip access-group 10 in
```

**Key Points:**
- **Numbers:** 1-99 only
- **Filters by:** Source IP only
- **Placement:** Close to destination
- **Implicit deny:** Ends with "deny any any"
- **Order matters:** Processed top-down. If acl rule is found in the first line then it won't read next lines.
- **permit any** it is compulsory to write permit any at the end because without it it wont allow anyother network

**Verification Commands:**




```cisco
show access-lists              ! View all ACLs with hit counts
show ip access-lists           ! View IP ACLs specifically
show running-config | include access-list  ! Find ACLs in config
show ip interface <INTERFACE>  ! See applied ACLs
```

# DHCP (Dynamic Host Configuration Protocol)

## What DHCP Does

DHCP (Dynamic Host Configuration Protocol) automatically assigns:
- IP address
- Subnet mask
- Default gateway
- DNS server

to hosts, eliminating manual IP configuration and reducing errors.

## DHCP Pool Setting

It's best to add DHCP pools via GUI in Cisco Packet Tracer as it is much easier.

## DHCP Relay (ip helper-address)

**Used when:**
- DHCP server is in a different subnet
- Router forwards DHCP broadcasts as unicasts

**Configuration:**

```cisco
interface <CLIENT_INTERFACE>
 ip helper-address <DHCP_SERVER_IP>
```

## Key Points (Very Important)

- DHCP server must have routing to all client subnets
- `ip helper-address` is mandatory across subnets
- Always exclude static IPs before creating pool

---
