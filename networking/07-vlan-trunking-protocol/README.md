# Networking Lab 07 — VLAN Trunking Protocol (VTP)

## Objective

Configure VTP across three switches and verify how Server, Client, and Transparent modes handle VLAN database changes.

## Environment

* **Tool:** Cisco Packet Tracer
* **Switches:** SW1, SW2, and SW3
* **Topology:** Three switches connected in a line (SW1 — SW2 — SW3)
* **VTP domain:** `ccna-labs`
* **VTP password:** `cisco`

## Scenario

The switches share one VTP domain. SW1 acts as the VTP Server, SW2 as a Client, and SW3 as a Transparent switch.

The goal is to create VLANs on the server, verify that the client learns them automatically, and confirm that the transparent switch keeps its own VLAN database.

## 1. Configure the VTP Domain and Password

On all three switches, configure the same domain and password:

```text
vtp domain ccna-labs
vtp password cisco
```

## 2. Assign VTP Modes

Configure each switch with its assigned mode.

### SW1 — Server

```text
vtp mode server
```

### SW2 — Client

```text
vtp mode client
```

### SW3 — Transparent

```text
vtp mode transparent
```

The mode determines how each switch handles VLAN database changes:
- **Server:** Can create, modify, and delete VLANs, and advertises those changes through VTP.
- **Client:** Learns and applies VLAN database changes received from a VTP server.
- **Transparent:** Maintains its own VLAN database and does not synchronize it with the VTP domain.

## 3. Configure Trunk Links

The switches are connected in a line, so both ends of each inter-switch link must be configured as trunks:

- SW1 ↔ SW2
- SW2 ↔ SW3

On each interface connected to another switch:

```text
interface <inter-switch-interface>
switchport mode trunk
```

A trunk carries traffic for multiple VLANs and allows VTP advertisements to travel between connected switches.

## 4. Create VLANs on the VTP Server

On SW1, create VLAN 10 and VLAN 20:

```text
vlan 10
name Sales
exit
vlan 20
name HR
exit
```

SW1 is the VTP Server, so its VLAN database changes are advertised to switches in the same VTP domain.

## 5. Verify VLAN Synchronization

Use the following commands to inspect the VLAN database and VTP configuration:

```text
show vlan brief
show vtp status
```

**SW1:** VLAN 10 (Sales) and VLAN 20 (HR) appeared in its VLAN database.

**SW2:** VLANs 10 and 20 appeared automatically after receiving the VTP updates from SW1. No manual creation was needed on the client.

**SW3:** VLANs 10 and 20 did not appear in its local VLAN database because Transparent mode does not apply VTP database updates.

A transparent switch can still receive and, depending on the VTP version, forward VTP advertisements; it simply does not synchronize its own VLAN database with them.

## 6. Create a Local VLAN on the Transparent Switch

On SW3, create VLAN 99:

```text
vlan 99
name Mgmt
exit
```

Verify the result with:

```text
show vlan brief
```

VLAN 99 appeared in SW3's local VLAN database. It was not propagated to SW1 or SW2 because VLAN changes made on a VTP Transparent switch are local.

## What I Learned

* How VTP domains group switches for VLAN database updates.
* The difference between VTP Server, Client, and Transparent modes.
* Why trunk links are required between switches for VTP advertisements.
* How a VTP Client learns VLANs created on a Server.
* How a Transparent switch maintains its own VLAN database instead of synchronizing it with the domain.
* How to verify VTP configuration and VLAN databases using Cisco IOS commands.

## Key Takeaway

VTP can distribute VLAN database changes from a Server to Clients in the same domain. A Transparent switch keeps its VLAN configuration independent, allowing local VLANs without synchronizing them across the VTP domain.
