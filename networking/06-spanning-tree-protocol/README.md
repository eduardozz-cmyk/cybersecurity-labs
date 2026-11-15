# Networking Lab 06 — Spanning Tree Protocol (STP)

## Objective

Observe how Spanning Tree Protocol prevents Layer 2 switching loops by selecting a Root Bridge, assigning port roles, and blocking redundant paths. Test how STP reacts when network links fail.

## Environment

* **Tool:** Cisco Packet Tracer
* **Switches:** SW1, SW2, and SW3
* **Topology:** Three switches connected in a triangle
* **VLAN:** VLAN 1
* **Protocol:** STP (enabled by default on the Cisco switches)

## Scenario

The three switches are connected in a triangle, creating a redundant Layer 2 path. Without a loop-prevention mechanism, Ethernet frames such as broadcast frames could circulate through the switches repeatedly.

STP calculates a loop-free logical topology. It elects one switch as the Root Bridge, selects a Root Port on each non-root switch, identifies Designated Ports for network segments, and blocks ports that would otherwise create a loop.

No manual STP configuration was required for this lab because STP was already enabled by default.

## 1. Verify STP

Run the following commands on each switch.

### On SW1, SW2, and SW3

    show spanning-tree
    show spanning-tree vlan 1

The commands display the spanning-tree information for VLAN 1, including the Root Bridge ID, local Bridge ID, port roles, port states, and path costs.

## 2. Identify the Root Bridge

The outputs from all three switches showed the same Root ID:

    Priority    32769
    Address     000C.CFA6.6209

On SW3, the output also displayed:

    This bridge is the root

Therefore, SW3 was elected as the Root Bridge.

All three switches had the same priority value, so the MAC address determined the election. SW3 had the lowest MAC address.

| Switch | Bridge MAC address | Role |
|---|---|---|
| SW1 | 0090.2B77.27D4 | Non-root switch |
| SW2 | 00E0.8FEB.4ECD | Non-root switch |
| SW3 | 000C.CFA6.6209 | Root Bridge |

The Root Bridge is the reference point STP uses when calculating the loop-free topology. Non-root switches select their best path toward it.

## 3. Observe Port Roles and States

The initial command output showed these port roles:

| Switch | Interface | Role | State |
|---|---|---|---|
| SW1 | Fa0/1 | Designated | Forwarding |
| SW1 | Fa0/2 | Root | Forwarding |
| SW2 | Fa0/1 | Alternate | Blocking |
| SW2 | Fa0/2 | Root | Forwarding |
| SW3 | Fa0/1 | Designated | Forwarding |
| SW3 | Fa0/2 | Designated | Forwarding |

### Root Port

A Root Port is the port on a non-root switch that provides its best path toward the Root Bridge. Each non-root switch has one Root Port per spanning-tree instance.

SW1 and SW2 each had a Root Port pointing toward the Root Bridge, SW3.

### Designated Port

A Designated Port is the forwarding port selected for a network segment. In this topology, SW3 was the Root Bridge, so both of its connected ports were Designated and Forwarding.

### Alternate Port

An Alternate Port provides another path toward the Root Bridge. STP keeps it blocked when using it would create a Layer 2 loop.

SW2 Fa0/1 was initially an Alternate Port in the Blocking state. SW2 continued to reach the Root Bridge through its Root Port, Fa0/2.

### Port States

* **Forwarding:** The port can forward frames.
* **Blocking:** The port does not forward normal user traffic, preventing a Layer 2 loop.

## 4. Test the Blocked Link

The initially blocked port was SW2 Fa0/1, connected to SW1.

I disconnected the cable between SW1 and SW2.

The network connectivity remained unchanged because the disconnected link was already blocked and was not forwarding normal user traffic. SW2 still had its active path toward the Root Bridge through Fa0/2.

This demonstrated that removing a blocked redundant link does not interrupt the active forwarding path.

I then reconnected the cable between SW1 and SW2. STP detected the restored topology and SW2 Fa0/1 returned to its blocked role.

## 5. Test STP Failover

With the triangle topology restored and SW2 Fa0/1 blocked, I disconnected the active link between SW1 and SW3.

This changed the active topology. STP recalculated the paths, and SW2 Fa0/1 became a forwarding port.

The previously blocked link could now be used to maintain connectivity through the remaining switches.

This demonstrated how STP can activate an alternative path when an active link fails.

## 6. Key STP Concepts

* **Root Bridge:** The switch with the lowest Bridge ID, used as the reference point for the spanning-tree topology.
* **Root Port:** The best path from a non-root switch toward the Root Bridge.
* **Designated Port:** The forwarding port selected for a network segment.
* **Alternate Port:** A backup path that STP blocks when necessary to prevent a loop.
* **Convergence:** The process in which STP recalculates the topology after a change, such as a link failure.

A Layer 2 loop can cause broadcast frames to circulate repeatedly because Ethernet frames do not have a TTL field. The redundant physical connections create the possibility of a loop; the absence of a TTL field means frames do not automatically expire as they circulate.

STP prevents this by creating a loop-free forwarding topology while retaining redundant links that can be used if the active topology changes.

## What I Learned

* How STP elects a Root Bridge.
* How the Bridge ID and MAC address affect the Root Bridge election when priorities are equal.
* The difference between Root, Designated, and Alternate port roles.
* Why STP blocks redundant paths.
* Why removing an already blocked link does not interrupt active traffic.
* How STP recalculates the topology and can unblock an alternative path after an active link fails.
* Why Layer 2 loops can cause broadcast storms and duplicate frames.

## Key Takeaway

STP prevents Layer 2 switching loops by creating a loop-free forwarding topology. It blocks redundant paths when necessary but keeps the network able to use an alternative path if an active link fails.
