# Networking Lab 03 — Basic IPv6 Configuration

## Objective

Configure IPv6 addressing on two routers and two PCs, enable IPv6 routing, add static routes between two LANs, and verify connectivity across the network.

## Environment

* **Tool:** Cisco Packet Tracer
* **Routers:** R1 and R2
* **Switches:** Two
* **PCs:** PC1 and PC2
* **Topology:** Two LANs connected through two routers
* **Routing method:** Static IPv6 routing

## Scenario

The topology contains two LANs. R1 connects to LAN 1, R2 connects to LAN 2, and the routers are connected through a separate point-to-point network.

The lab uses the documentation prefix 2001:DB8::/32 for IPv6 addressing examples. Each LAN and the router-to-router link use a separate /64 network.

## 1. IPv6 Addressing Plan

I used three /64 networks:

| Network | IPv6 Prefix |
|---|---|
| LAN 1 | 2001:DB8:1::/64 |
| LAN 2 | 2001:DB8:2::/64 |
| R1–R2 link | 2001:DB8:3::/64 |

### Router Interface Addressing

| Router | Interface | IPv6 Address | Connected Network |
|---|---|---|---|
| R1 | G0/1 | 2001:DB8:1::1/64 | LAN 1 |
| R1 | G0/0 | 2001:DB8:3::1/64 | R1–R2 link |
| R2 | G0/1 | 2001:DB8:2::1/64 | LAN 2 |
| R2 | G0/0 | 2001:DB8:3::2/64 | R1–R2 link |

Both router interfaces on the inter-router link use addresses from the same /64 network.

### PC Addressing

| Device | IPv6 Address | Prefix Length | Default Gateway |
|---|---|---:|---|
| PC1 | 2001:DB8:1::10 | /64 | 2001:DB8:1::1 |
| PC2 | 2001:DB8:2::10 | /64 | 2001:DB8:2::1 |

The default gateway on each PC is the IPv6 address of the router interface connected to its LAN.

## 2. Router IPv6 Configuration

IPv6 routing was enabled globally on both routers.

    ipv6 unicast-routing

The router interfaces were assigned their IPv6 addresses and enabled.

### R1

    interface g0/1
    ipv6 address 2001:DB8:1::1/64
    no shutdown
    exit

    interface g0/0
    ipv6 address 2001:DB8:3::1/64
    no shutdown
    exit

### R2

    interface g0/1
    ipv6 address 2001:DB8:2::1/64
    no shutdown
    exit

    interface g0/0
    ipv6 address 2001:DB8:3::2/64
    no shutdown
    exit

The ipv6 unicast-routing command must be configured on both routers. Assigning IPv6 addresses and adding routes does not by itself enable a router to forward IPv6 packets between interfaces.

## 3. Static IPv6 Routing

Each router automatically knows its directly connected IPv6 networks. Static routes were added to provide a path to the LAN connected to the other router.

### R1

LAN 2 is reachable through R2 at 2001:DB8:3::2.

    ipv6 route 2001:DB8:2::/64 2001:DB8:3::2

### R2

LAN 1 is reachable through R1 at 2001:DB8:3::1.

    ipv6 route 2001:DB8:1::/64 2001:DB8:3::1

Both routers need a route to the remote LAN so communication can work in both directions.

## 4. Verification and Testing

I used the following commands to verify the IPv6 configuration.

### Check interface addressing and status

    show ipv6 interface brief

This command displays IPv6 addresses and interface status.

### Check the IPv6 routing table

    show ipv6 route

The routing table uses these route codes:

* C — Directly connected network.
* L — Local IPv6 address assigned to a router interface.
* S — Static route configured manually.

The routing tables showed the directly connected LAN and router-link networks, along with a static route to the remote LAN on each router.

### Test connectivity

* Confirmed that both PCs could ping their local default gateways.
* Confirmed connectivity between the two router interfaces.
* Confirmed that each router could reach the LAN interface of the remote router.
* From PC1, successfully pinged R2's LAN interface.

## 5. Troubleshooting

The initial end-to-end connectivity test failed because IPv6 routing had not been enabled on R2.

I had configured ipv6 unicast-routing on R1 but had forgotten to configure it on R2.

After enabling IPv6 routing on R2, the router had the required global IPv6 forwarding configuration.

This demonstrated that correct interface addresses and static routes are not sufficient if IPv6 packet forwarding is not enabled on the router.

## What I Learned

* How IPv6 addresses are structured using eight hexadecimal groups.
* How to compress IPv6 addresses by removing leading zeros and replacing consecutive zero groups with ::.
* Why :: can only appear once in an IPv6 address.
* How to assign IPv6 addresses and prefix lengths to router interfaces.
* How to enable IPv6 routing with ipv6 unicast-routing.
* How to configure static IPv6 routes using a destination prefix and next-hop address.
* How to configure IPv6 addresses and default gateways on PCs.
* How to read connected, local, and static IPv6 routes in show ipv6 route.
* Why IPv6 routing must be enabled on every router that needs to forward IPv6 traffic.

## Key Takeaway

IPv6 static routing follows the same basic logic as IPv4 static routing, but uses IPv6 prefixes and next-hop addresses. A router needs both a valid route to the destination and IPv6 forwarding enabled to move packets between networks.
