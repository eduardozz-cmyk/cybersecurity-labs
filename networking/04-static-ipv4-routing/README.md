# Networking Lab 04 — Static IPv4 Routing

## Objective

Configure IPv4 addressing on three routers and two PCs, connect two LANs through two router-to-router links, configure static routes, and verify end-to-end connectivity.

## Environment

* **Tool:** Cisco Packet Tracer
* **Routers:** R1, R2, and R3
* **Switches:** Two
* **PCs:** PC1 and PC2
* **Topology:** Two LANs connected through three routers
* **Routing method:** Static IPv4 routing

## Scenario

The topology contains two LANs connected through three routers.

R1 connects LAN 1 to R2. R2 connects R1 to R3 through two separate router-to-router links. R3 connects to LAN 2.

Each LAN and each router-to-router link uses a separate IPv4 network. Static routes are required so that every router knows how to reach the remote LANs.

## 1. IPv4 Addressing Plan

I used four /24 networks:

| Network | Network Address | Purpose |
|---|---|---|
| LAN 1 | 192.168.1.0/24 | PC1 and R1 |
| R1–R2 link | 192.168.2.0/24 | First router-to-router link |
| R2–R3 link | 192.168.3.0/24 | Second router-to-router link |
| LAN 2 | 192.168.4.0/24 | R3 and PC2 |

### Router Interface Addressing

| Router | Interface | IP Address | Connected Network |
|---|---|---|---|
| R1 | G0/0 | 192.168.1.1/24 | LAN 1 |
| R1 | G0/1 | 192.168.2.1/24 | R1–R2 link |
| R2 | G0/0 | 192.168.2.2/24 | R1–R2 link |
| R2 | G0/1 | 192.168.3.2/24 | R2–R3 link |
| R3 | G0/0 | 192.168.3.1/24 | R2–R3 link |
| R3 | G0/1 | 192.168.4.1/24 | LAN 2 |

Each router-to-router link uses a separate subnet. The two interfaces at opposite ends of each link belong to the same subnet.

### PC Addressing

| Device | IP Address | Subnet Mask | Default Gateway |
|---|---|---|---|
| PC1 | 192.168.1.10 | 255.255.255.0 | 192.168.1.1 |
| PC2 | 192.168.4.10 | 255.255.255.0 | 192.168.4.1 |

Each PC uses the IP address of the router interface connected to its own LAN as its default gateway.

## 2. Router Interface Configuration

I configured the IP address on each router interface according to the addressing plan and enabled the interfaces with no shutdown.

### R1

    interface g0/0
    ip address 192.168.1.1 255.255.255.0
    no shutdown
    exit

    interface g0/1
    ip address 192.168.2.1 255.255.255.0
    no shutdown
    exit

### R2

    interface g0/0
    ip address 192.168.2.2 255.255.255.0
    no shutdown
    exit

    interface g0/1
    ip address 192.168.3.2 255.255.255.0
    no shutdown
    exit

### R3

    interface g0/0
    ip address 192.168.3.1 255.255.255.0
    no shutdown
    exit

    interface g0/1
    ip address 192.168.4.1 255.255.255.0
    no shutdown
    exit

## 3. Static IPv4 Routing

Each router automatically learns the networks directly connected to its interfaces. Static routes are needed for networks that are not directly connected.

The static route command uses this format:

    ip route [destination-network] [subnet-mask] [next-hop-address]

The destination must be the network address, not an individual host address. The next hop must be the IP address of the neighboring router interface that leads toward the destination.

### R1

R1 needs a route to LAN 2 through R2.

    ip route 192.168.4.0 255.255.255.0 192.168.2.2

### R2

R2 needs routes to both LANs.

Route to LAN 1 through R1:

    ip route 192.168.1.0 255.255.255.0 192.168.2.1

Route to LAN 2 through R3:

    ip route 192.168.4.0 255.255.255.0 192.168.3.1

### R3

R3 needs a route to LAN 1 through R2.

    ip route 192.168.1.0 255.255.255.0 192.168.3.2

Static routes are required in both directions. Each router must have a route toward the remote LANs so that request and reply packets can travel between the PCs.

## 4. Verification and Testing

### Check interface addresses and status

    show ip interface brief

This command displays the IP addresses and operational status of router interfaces.

### Check the routing table

    show ip route

The routing table uses these relevant route codes:

* C — Directly connected network.
* L — Local IP address assigned to a router interface.
* S — Static route configured manually.

I verified that each router had its directly connected networks and the required static routes to remote LANs.

### Test connectivity

* Verified connectivity between neighboring router interfaces.
* Verified that PC1 and PC2 could ping their respective default gateways.
* Successfully pinged PC2 from PC1.
* Used tracert from PC1 to PC2 to inspect the path taken by packets.

### Traceroute result

Command executed on PC1:

    tracert 192.168.4.10

The trace showed the following path:

| Hop | IP Address | Device |
|---|---|---|
| 1 | 192.168.1.1 | R1 |
| 2 | 192.168.2.2 | R2 |
| 3 | 192.168.3.1 | R3 |
| 4 | 192.168.4.10 | PC2 |

This confirmed that packets traveled from PC1 through R1, R2, and R3 before reaching PC2.

## 5. Troubleshooting

The initial end-to-end ping did not succeed because R3 did not have a route to LAN 1.

R3 knew its directly connected networks, including the R2–R3 link and LAN 2, but it did not know how to reach 192.168.1.0/24.

I added a static route on R3 pointing to R2's interface on the R2–R3 link:

    ip route 192.168.1.0 255.255.255.0 192.168.3.2

After adding this route, PC1 was able to ping PC2 successfully.

This demonstrated that successful communication requires a valid route in both directions. A router must know where to forward packets for remote networks; knowing only its directly connected networks is not enough.

## What I Learned

* How to identify router interfaces by checking which devices and networks they connect to.
* How to configure IPv4 addresses and subnet masks on router interfaces.
* Why each separate LAN and router-to-router link needs a unique, non-overlapping network.
* How to configure static routes using a destination network, subnet mask, and next-hop address.
* Why the destination in a static route is the network address rather than an individual host address.
* How to determine the correct next hop based on the direction toward the destination network.
* How to read connected, local, and static routes in the IPv4 routing table.
* Why static routes must provide a path in both directions for end-to-end communication.
* How to use ping and tracert to verify connectivity and inspect the packet path.

## Key Takeaway

Static routing requires each router to know how to reach remote networks. In a multi-router topology, routes must be configured on every router that needs to forward traffic toward a remote destination, including the return path. The next-hop address must point to the neighboring router that leads toward that destination.
