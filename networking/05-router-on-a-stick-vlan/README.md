# Networking Lab 05 — Router-on-a-Stick VLAN Configuration

## Objective

Create two separate VLANs on a switch, configure a trunk connection to a router, and use router subinterfaces to enable communication between the VLANs through one physical router interface.

## Environment

* **Tool:** Cisco Packet Tracer
* **Router:** R1
* **Switch:** SW1
* **PCs:** PC1, PC2, and PC3
* **Topology:** One router connected to one switch, with three PCs
* **Routing method:** Router-on-a-Stick (inter-VLAN routing)

## Scenario

The network contains two VLANs:

* VLAN 10 — HR
* VLAN 20 — Sales

PC1 and PC3 belong to VLAN 10. PC2 belongs to VLAN 20. Devices in different VLANs are in separate IP networks, so they need a Layer 3 device to communicate.

The switch connects to the router through one physical link configured as a trunk. On the router, two logical subinterfaces handle the traffic for the two VLANs. This setup is known as Router-on-a-Stick.

## 1. VLAN and IP Addressing Plan

### VLANs and Switch Ports

| VLAN | Name | Switch Port | Connected PC |
|---|---|---|---|
| 10 | HR | Fa0/1 | PC1 |
| 20 | Sales | Fa0/2 | PC2 |
| 10 | HR | Fa0/3 | PC3 |

An access port belongs to one VLAN. The switch associates untagged traffic received on that port with its configured VLAN.

### PC Addressing

| Device | IP Address | Subnet Mask | Default Gateway | VLAN |
|---|---|---|---|---|
| PC1 | 192.168.10.10 | 255.255.255.0 | 192.168.10.1 | 10 |
| PC2 | 192.168.20.10 | 255.255.255.0 | 192.168.20.1 | 20 |
| PC3 | 192.168.10.11 | 255.255.255.0 | 192.168.10.1 | 10 |

### Router Subinterfaces

| Router Interface | VLAN | IP Address | Purpose |
|---|---|---|---|
| G0/0.10 | 10 | 192.168.10.1/24 | Default gateway for VLAN 10 |
| G0/0.20 | 20 | 192.168.20.1/24 | Default gateway for VLAN 20 |

The physical router interface G0/0 provides the connection to the switch. It does not need its own IP address in this configuration. The IP addresses are assigned to the subinterfaces.

## 2. Create VLANs and Configure Access Ports

### On SW1 (Switch)

Create VLAN 10 and VLAN 20 and give them names:

    configure terminal
    vlan 10
    name HR
    vlan 20
    name Sales

Assign the access ports to their respective VLANs:

    interface fa0/1
    switchport mode access
    switchport access vlan 10
    exit

    interface fa0/2
    switchport mode access
    switchport access vlan 20
    exit

    interface fa0/3
    switchport mode access
    switchport access vlan 10
    exit

Verify the VLAN and port assignments:

    show vlan brief

The output showed Fa0/1 and Fa0/3 assigned to VLAN 10, and Fa0/2 assigned to VLAN 20.

## 3. Configure the Trunk Connection

The switch connects to the router through SW1 G0/1 and R1 G0/0.

### On SW1 (Switch)

Configure the switch port connected to the router as a trunk:

    configure terminal
    interface gigabitEthernet 0/1
    switchport mode trunk

A trunk carries traffic for multiple VLANs over one physical link. This allows VLAN 10 and VLAN 20 traffic to share the connection between the switch and router.

The trunk uses IEEE 802.1Q (dot1Q) tagging to identify the VLAN associated with each Ethernet frame.

### On R1 (Router)

Enable the physical interface connected to the switch:

    configure terminal
    interface g0/0
    no shutdown

The router interface must be enabled for its subinterfaces to operate. The physical interface itself does not need an IP address for this Router-on-a-Stick setup.

## 4. Configure Router Subinterfaces

A subinterface is a logical interface created under a physical router interface. Each subinterface is associated with a VLAN and has an IP address that serves as that VLAN's default gateway.

### On R1 (Router)

Configure the subinterface for VLAN 10:

    interface g0/0.10
    encapsulation dot1Q 10
    ip address 192.168.10.1 255.255.255.0
    exit

Configure the subinterface for VLAN 20:

    interface g0/0.20
    encapsulation dot1Q 20
    ip address 192.168.20.1 255.255.255.0
    exit

The command encapsulation dot1Q associates a subinterface with a VLAN ID. For example, encapsulation dot1Q 10 makes G0/0.10 handle traffic tagged for VLAN 10.

The two subinterfaces use different IP networks and act as the default gateways for their respective VLANs.

## 5. Configure PC Addresses

Configure each PC through Desktop → IP Configuration in Packet Tracer.

### On PC1

* IP address: 192.168.10.10
* Subnet mask: 255.255.255.0
* Default gateway: 192.168.10.1

### On PC2

* IP address: 192.168.20.10
* Subnet mask: 255.255.255.0
* Default gateway: 192.168.20.1

### On PC3

* IP address: 192.168.10.11
* Subnet mask: 255.255.255.0
* Default gateway: 192.168.10.1

## 6. How Router-on-a-Stick Works

The switch and router use different configurations on the two ends of the same physical connection:

* SW1 G0/1 is configured as a trunk so it can carry traffic from multiple VLANs.
* R1 G0/0 is the physical router interface connected to that trunk.
* R1 G0/0.10 handles VLAN 10 traffic.
* R1 G0/0.20 handles VLAN 20 traffic.

The router uses the 802.1Q VLAN tag to identify which subinterface should process an incoming frame. When traffic needs to travel to another VLAN, the router routes the packet to the destination network and sends it back through the appropriate subinterface.

For communication within the same VLAN, such as PC1 and PC3, the switch can forward traffic at Layer 2 without routing it through the router.

For communication between different VLANs, such as PC1 and PC2, the source PC recognizes that the destination is outside its local subnet and sends the traffic to its default gateway. The router then forwards it toward the destination VLAN.

This allows multiple separate IP networks to use one physical router connection without assigning a separate physical router interface to each VLAN.

## 7. Verification and Testing

### On R1 (Router)

Check the physical interface and subinterface addresses and status:

    show ip interface brief

Both G0/0.10 and G0/0.20 were up.

Check the routing table:

    show ip route

The router should have directly connected routes for 192.168.10.0/24 and 192.168.20.0/24.

### On SW1 (Switch)

Check VLAN membership:

    show vlan brief

Check trunk status and the VLANs carried by the trunk:

    show interfaces trunk

The G0/1 link was trunking, using 802.1Q, with VLANs 10 and 20 active.

### From the PCs

I tested connectivity between the PCs and to both router gateway addresses.

* PC1 successfully communicated with PC3. Both PCs belong to VLAN 10, so their traffic can be switched within the same VLAN.
* PC2 successfully communicated with PC1 and PC3. These tests crossed between VLAN 20 and VLAN 10, confirming that inter-VLAN routing worked.
* All three PCs successfully pinged both router gateway addresses: 192.168.10.1 and 192.168.20.1.

These tests confirmed that the access-port assignments, trunk connection, router subinterfaces, default gateways, and inter-VLAN routing were working.

## 8. Router-on-a-Stick Limitations

All VLAN traffic between the switch and router shares one physical link. The available bandwidth of that link is therefore shared by the VLANs.

For example, if the connection is 1 Gbps, the combined traffic cannot exceed the link's practical capacity. Heavy traffic can create congestion and affect communication across VLANs.

The router must also process traffic routed between VLANs. Its forwarding capacity can become another bottleneck depending on the router model and workload.

In larger networks, a Layer 3 switch can perform inter-VLAN routing internally and may provide higher forwarding performance.

## What I Learned

* How to create VLANs and assign switch access ports to them.
* The difference between an access port and a trunk port.
* How a trunk carries multiple VLANs over one physical connection.
* How 802.1Q tags identify VLAN traffic on a trunk.
* How router subinterfaces associate one physical router interface with multiple VLANs.
* How encapsulation dot1Q binds a router subinterface to a VLAN ID.
* How each subinterface can provide the default gateway for a separate IP network.
* Why devices in different VLANs need Layer 3 routing to communicate.
* How Router-on-a-Stick allows inter-VLAN routing without using a separate physical router interface for every VLAN.
* Why sharing one physical link can create a bandwidth bottleneck as traffic increases.

## Key Takeaway

VLANs separate devices into different Layer 2 networks. A trunk carries traffic from multiple VLANs over one physical connection, and router subinterfaces use 802.1Q tagging to identify and route traffic between those networks. Router-on-a-Stick provides inter-VLAN routing with one physical router interface, but all traffic on that connection shares its bandwidth.
