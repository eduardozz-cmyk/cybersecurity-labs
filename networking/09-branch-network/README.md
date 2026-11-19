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

End-device switch ports are access ports assigned to their corresponding VLAN. The switch port connected to each router is a trunk so it can carry all three VLANs.

### On both switches

Create the VLANs:

```text
configure terminal
vlan 10
 name Management
exit
vlan 20
 name Sales
exit
vlan 30
 name HR
exit
```

Assign each end-device port to its VLAN. Replace the interface names with the ports connected to the devices:

```text
interface <server-port>
 switchport mode access
 switchport access vlan 10
exit
interface <sales-pc-port>
 switchport mode access
 switchport access vlan 20
exit
interface <hr-pc-port>
 switchport mode access
 switchport access vlan 30
exit
```

Configure the switch port connected to the router as a trunk:

```text
interface <router-facing-port>
 switchport mode trunk
exit
end
```

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

## 3. Configure Router-on-a-Stick

The physical router interface connected to the switch must be enabled. Create one subinterface for each VLAN, associate it with the VLAN ID using 802.1Q encapsulation, and assign its gateway address.

Replace `<LAN-interface>` with the physical router interface connected to the switch. The example uses G0/0.

### On R1 — Site A

```text
configure terminal
interface gigabitEthernet 0/0
 no shutdown
exit

interface gigabitEthernet 0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
exit
interface gigabitEthernet 0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
exit
interface gigabitEthernet 0/0.30
 encapsulation dot1Q 30
 ip address 192.168.30.1 255.255.255.0
exit
```

### On R2 — Site B

```text
configure terminal
interface gigabitEthernet 0/0
 no shutdown
exit

interface gigabitEthernet 0/0.10
 encapsulation dot1Q 10
 ip address 192.168.110.1 255.255.255.0
exit
interface gigabitEthernet 0/0.20
 encapsulation dot1Q 20
 ip address 192.168.120.1 255.255.255.0
exit
interface gigabitEthernet 0/0.30
 encapsulation dot1Q 30
 ip address 192.168.130.1 255.255.255.0
exit
```

The same VLAN ID at each site does not mean the sites share one Layer 2 VLAN. Each site has its own local VLAN and IP subnet; communication between sites is routed through R1 and R2.

## 4. Configure the WAN Link

Configure the router interfaces connected to each other. Replace the example interface name with the actual WAN-facing interface in the topology.

### On R1

```text
configure terminal
interface <WAN-interface>
 ip address 10.0.0.1 255.255.255.252
 no shutdown
exit
```

### On R2

```text
configure terminal
interface <WAN-interface>
 ip address 10.0.0.2 255.255.255.252
 no shutdown
exit
end
```

Both interfaces belong to the same point-to-point subnet, allowing the routers to communicate over the WAN link.

## 5. Configure Static Routes

Each router already knows its directly connected VLAN networks and the WAN network. Static routes are needed to reach the VLAN subnets at the other site.

The command format is:

```text
ip route <destination-network> <subnet-mask> <next-hop>
```

### On R1 — Routes to Site B

```text
configure terminal
ip route 192.168.110.0 255.255.255.0 10.0.0.2
ip route 192.168.120.0 255.255.255.0 10.0.0.2
ip route 192.168.130.0 255.255.255.0 10.0.0.2
```

### On R2 — Routes to Site A

```text
configure terminal
ip route 192.168.10.0 255.255.255.0 10.0.0.1
ip route 192.168.20.0 255.255.255.0 10.0.0.1
ip route 192.168.30.0 255.255.255.0 10.0.0.1
end
```

Both routers need routes to the remote VLANs. The return routes are necessary so replies can travel back to the source network.

## 6. Configure End Devices

Configure each PC and server through Desktop → IP Configuration in Packet Tracer.

For every device, set:
- An unused IP address in its VLAN subnet.
- Subnet mask: `255.255.255.0`.
- Default gateway: the IP address of its local router subinterface.

For example, a PC in Site A's VLAN 20 uses an address in `192.168.20.0/24` and gateway `192.168.20.1`. A server in Site B's VLAN 10 uses an address in `192.168.110.0/24` and gateway `192.168.110.1`.

## 7. Verification and Testing

Useful commands:

### On switches

```text
show vlan brief
show interfaces trunk
```

### On routers

```text
show ip interface brief
show ip route
show running-config
```

Use `ping <destination-IP>` from PCs and servers to test connectivity.

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
