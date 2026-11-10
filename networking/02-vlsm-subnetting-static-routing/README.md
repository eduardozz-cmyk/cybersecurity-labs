# Networking Lab 02 — VLSM Subnetting and Static Routing

## Objective

Reinforce VLSM subnetting and static routing by designing and configuring a network with four LANs connected through two routers.

## Environment

* **Tool:** Cisco Packet Tracer
* **Routers:** R1 and R2
* **Topology:** Four LANs connected through two routers
* **Router connection:** Serial point-to-point link
* **Routing method:** Static routing

## Scenario

The starting network was 192.168.10.0/24. It needed to support four LANs with different host requirements and a point-to-point connection between R1 and R2.

## 1. VLSM Subnet Allocation

I allocated the subnets from largest to smallest host requirement.

| Network | Required Hosts | Prefix | Subnet Mask | Network Address |
|---|---:|---:|---|---|
| LAN A | 100 | /25 | 255.255.255.128 | 192.168.10.0 |
| LAN B | 50 | /26 | 255.255.255.192 | 192.168.10.128 |
| LAN C | 20 | /27 | 255.255.255.224 | 192.168.10.192 |
| LAN D | 2 | /30 | 255.255.255.252 | 192.168.10.224 |
| R1–R2 link | 2 | /30 | 255.255.255.252 | 192.168.10.228 |

### Address Ranges

| Network | First Usable | Last Usable | Broadcast |
|---|---|---|---|
| LAN A | 192.168.10.1 | 192.168.10.126 | 192.168.10.127 |
| LAN B | 192.168.10.129 | 192.168.10.190 | 192.168.10.191 |
| LAN C | 192.168.10.193 | 192.168.10.222 | 192.168.10.223 |
| LAN D | 192.168.10.225 | 192.168.10.226 | 192.168.10.227 |
| R1–R2 link | 192.168.10.229 (R1) | 192.168.10.230 (R2) | 192.168.10.231 |

## 2. Network Configuration

* Connected LAN A and LAN B to R1.
* Connected LAN C and LAN D to R2.
* Connected R1 and R2 through a serial link.
* Configured router interfaces with addresses from their respective subnets.
* Configured each PC with an IP address, subnet mask, and default gateway.
* Verified that each PC could reach its default gateway.

## 3. Static Routing

Static routes were configured so each router could reach the LANs connected to the other router.

### R1

LAN C and LAN D are reachable through R2 at 192.168.10.230.

    ip route 192.168.10.192 255.255.255.224 192.168.10.230
    ip route 192.168.10.224 255.255.255.252 192.168.10.230

### R2

LAN A and LAN B are reachable through R1 at 192.168.10.229.

    ip route 192.168.10.0 255.255.255.128 192.168.10.229
    ip route 192.168.10.128 255.255.255.192 192.168.10.229

Both routers require routes to the remote LANs so traffic can travel in both directions.

## 4. Verification and Testing

I used the following commands to verify the configuration.

### Check interface status and addressing

    show ip interface brief

### Check the routing table

    show ip route

The routing table uses these route codes:

* C — Directly connected network.
* L — Local IP address assigned to a router interface.
* S — Static route configured manually.

### Test connectivity

* Verified connectivity between the router interfaces.
* Confirmed that each PC could ping its default gateway.
* Tested connectivity between PCs on different LANs.
* Confirmed successful communication between all four PCs in both directions.

All connectivity tests succeeded.

## What I Learned

* Reinforced VLSM calculations with different host requirements.
* Practiced allocating subnet ranges without overlapping networks.
* Configured router interfaces and a serial point-to-point connection.
* Improved my understanding of static routes and next-hop addresses.
* Verified bidirectional connectivity using ping and the routing table.

## Key Takeaway

Static routing requires each router to know how to reach remote networks. Both the forward path and the return path must have valid routes for end-to-end communication.
