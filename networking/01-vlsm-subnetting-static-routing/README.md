# Networking Lab 01 — VLSM Subnetting and Static Routing

## Objective

Divide a 192.168.1.0/24 network into smaller subnets using Variable Length Subnet Masking (VLSM), configure router and PC addressing, implement static routes between two routers, and verify end-to-end connectivity.

## Environment

* **Tool:** Cisco Packet Tracer
* **Routers:** R1 and R2
* **Topology:** Four LANs connected through two routers
* **Routing method:** Static routing

## Scenario

The starting network was 192.168.1.0/24. It needed to support four LANs with different host requirements and a point-to-point connection between R1 and R2.

The lab required using the first usable address for a PC and the last usable address as the default gateway in each LAN.

## 1. VLSM Subnet Allocation

I allocated the subnets from largest to smallest host requirement to use the address space efficiently.

| Network | Required Hosts | Prefix | Subnet Mask | Network Address |
|---|---:|---:|---|---|
| LAN A | 60 | /26 | 255.255.255.192 | 192.168.1.0 |
| LAN B | 30 | /27 | 255.255.255.224 | 192.168.1.64 |
| LAN C | 14 | /28 | 255.255.255.240 | 192.168.1.96 |
| LAN D | 2 | /30 | 255.255.255.252 | 192.168.1.112 |
| R1–R2 link | 2 | /30 | 255.255.255.252 | 192.168.1.116 |

### Address Ranges

| Network | First Usable | Last Usable | Broadcast |
|---|---|---|---|
| LAN A | 192.168.1.1 | 192.168.1.62 | 192.168.1.63 |
| LAN B | 192.168.1.65 | 192.168.1.94 | 192.168.1.95 |
| LAN C | 192.168.1.97 | 192.168.1.110 | 192.168.1.111 |
| LAN D | 192.168.1.113 | 192.168.1.114 | 192.168.1.115 |
| R1–R2 link | 192.168.1.117 (R1) | 192.168.1.118 (R2) | 192.168.1.119 |

## 2. Router Interface Addressing

Each router interface was assigned an IP address belonging to the network connected to that interface.

### R1

| Interface | IP Address | Connected Network |
|---|---|---|
| G0/0 | 192.168.1.62/26 | LAN A |
| G0/1 | 192.168.1.94/27 | LAN B |
| Serial interface | 192.168.1.117/30 | R1–R2 link |

### R2

| Interface | IP Address | Connected Network |
|---|---|---|
| G0/0 | 192.168.1.110/28 | LAN C |
| G0/1 | 192.168.1.114/30 | LAN D |
| Serial interface | 192.168.1.118/30 | R1–R2 link |

The serial interfaces use addresses from the same /30 subnet. R1 uses 192.168.1.117 and R2 uses 192.168.1.118.

## 3. PC Addressing

Each PC was configured manually with:

* A unique IP address in its LAN subnet
* The subnet mask for that LAN
* The LAN router interface as its default gateway

The first usable address was used for the PC addressing plan, and the last usable address was reserved for the default gateway, as required by the lab.

## 4. Static Routing

The routers initially knew their directly connected networks. Static routes were added so each router could reach the LANs connected to the other router.

### R1

LAN C and LAN D are reachable through R2 at 192.168.1.118.

    ip route 192.168.1.96 255.255.255.240 192.168.1.118
    ip route 192.168.1.112 255.255.255.252 192.168.1.118


### R2

LAN A and LAN B are reachable through R1 at 192.168.1.117.

    ip route 192.168.1.0 255.255.255.192 192.168.1.117
    ip route 192.168.1.64 255.255.255.224 192.168.1.117


Both routers need routes because communication must work in both directions. A request may reach a remote LAN, but the reply also needs a route back to the source network.

## 5. Verification and Testing

I used the following commands during verification.

### Check interface addressing and status

    show ip interface brief


### Check the routing table

    show ip route


The routing table showed these route codes:

* C — Directly connected network.
* L — Local IP address assigned to a router interface.
* S — Static route configured manually.

### Test the router-to-router link

From R1, I pinged R2's serial address:

    ping 192.168.1.118


The ping succeeded, confirming connectivity between the router interfaces.

### Test inter-LAN connectivity

I also pinged PCs across different LANs. The tests succeeded, confirming that the PC addressing, default gateways, and static routes were working together.

## What I Learned

* How to divide a /24 network into different subnet sizes using VLSM.
* How to calculate network addresses, usable host ranges, and broadcast addresses.
* Why each router interface needs an address from its connected subnet.
* Why a /30 subnet is suitable for a point-to-point connection with two endpoints.
* How a PC uses its subnet mask and default gateway to reach another network.
* How a static route identifies a destination network and the next-hop router.
* Why both routers need routes for bidirectional communication.
* How to read the basic route codes in show ip route.

## Key Takeaway

A router automatically knows its directly connected networks, but it needs additional routing information to reach remote networks. In this lab, static routes provided that information by directing traffic to the next-hop router.
