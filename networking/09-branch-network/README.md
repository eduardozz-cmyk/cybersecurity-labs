# Networking Lab 09 — Branch Network Configuration

## Objective

Connect two separate sites using VLANs, Router-on-a-Stick, and static routing. Verify communication between devices in the same VLAN across different sites and between all VLANs.

## Environment

* **Tool:** Cisco Packet Tracer
* **Routers:** R1 (Site A) and R2 (Site B)
* **Switches:** One switch at each site
* **End devices:** Two PCs and one server at each site
* **WAN network:** `10.0.0.0/30`

## 1. VLAN and Device Assignment

Both switches use the same VLAN IDs, but each site's VLANs have separate IP subnets.

| VLAN | Purpose | Site A | Site B |
|---|---|---|---|
| 10 | Management | Server1 | Server2 |
| 20 | Sales | PC1 | PC3 |
| 30 | HR | PC2 | PC4 |

End-device switch ports are assigned to their corresponding access VLAN. The switch port connected to each router is configured as a trunk so it can carry all three VLANs.

## 2. IP Addressing Plan

### Site A — R1

| VLAN / Link | Network | Gateway / Interface IP |
|---|---|---|
| VLAN 10 | `192.168.10.0/24` | `192.168.10.1` |
| VLAN 20 | `192.168.20.0/24` | `192.168.20.1` |
| VLAN 30 | `192.168.30.0/24` | `192.168.30.1` |
| WAN | `10.0.0.0/30` | `10.0.0.1` |

### Site B — R2

| VLAN / Link | Network | Gateway / Interface IP |
|---|---|---|
| VLAN 10 | `192.168.110.0/24` | `192.168.110.1` |
| VLAN 20 | `192.168.120.0/24` | `192.168.120.1` |
| VLAN 30 | `192.168.130.0/24` | `192.168.130.1` |
| WAN | `10.0.0.0/30` | `10.0.0.2` |

The WAN link uses the `255.255.255.252` subnet mask. Each end device receives an unused IP address from its VLAN subnet, the `255.255.255.0` mask, and that VLAN's router subinterface address as its default gateway.

## 3. Configure VLANs and Router-on-a-Stick

Create VLANs 10, 20, and 30 on both switches and assign the end-device ports according to the VLAN assignment table.

Configure the router-facing switch port as a trunk. On each router, enable the physical interface connected to the switch and create one subinterface for each VLAN. Associate each subinterface with its VLAN ID using 802.1Q encapsulation and assign the gateway address from the IP plan.

The same VLAN ID at each site does not mean the sites share one Layer 2 VLAN. Each site has its own local VLAN and IP subnet; communication between sites is routed through R1 and R2.

## 4. Configure the WAN Link

Configure the router interfaces connected to each other using the WAN addresses:

* R1: `10.0.0.1/30`
* R2: `10.0.0.2/30`

Both interfaces belong to the same point-to-point subnet, allowing the routers to communicate over the WAN link.

## 5. Configure Static Routes

Each router already knows its directly connected VLAN networks and the WAN network. Static routes are needed to reach the VLAN subnets at the other site.

### On R1 — Routes to Site B

| Destination network | Subnet mask | Next hop |
|---|---|---|
| `192.168.110.0` | `255.255.255.0` | `10.0.0.2` |
| `192.168.120.0` | `255.255.255.0` | `10.0.0.2` |
| `192.168.130.0` | `255.255.255.0` | `10.0.0.2` |

### On R2 — Routes to Site A

| Destination network | Subnet mask | Next hop |
|---|---|---|
| `192.168.10.0` | `255.255.255.0` | `10.0.0.1` |
| `192.168.20.0` | `255.255.255.0` | `10.0.0.1` |
| `192.168.30.0` | `255.255.255.0` | `10.0.0.1` |

Both routers need routes to the remote VLANs. The return routes are necessary so replies can travel back to the source network.

## 6. Verification and Testing

I tested connectivity using ping:

* Devices within each site successfully reached their local default gateways.
* Devices in different VLANs at the same site successfully communicated, confirming inter-VLAN routing.
* Devices in corresponding VLANs across Site A and Site B successfully communicated.
* The two servers successfully communicated with each other.
* The servers successfully reached the PCs across both sites.

All tests were successful, confirming that the VLAN assignments, router subinterfaces, WAN link, and static routes were working.

## What I Learned

* How to combine VLANs and Router-on-a-Stick across two separate sites.
* Why the same VLAN ID at different sites does not automatically create one shared Layer 2 network.
* How a WAN link connects routers, while static routes tell them which remote networks are reachable through that link.
* Why both routers need routes so traffic can reach the destination and return to the source.

## Key Takeaway

A WAN link provides connectivity between the routers, but static routes provide the paths to the remote VLAN subnets. Successful communication requires correct local VLAN routing and a return route at the other site.
