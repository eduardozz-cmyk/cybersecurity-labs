# Networking Lab 08 — Network Address Translation (NAT)

## Objective

Configure and test Static NAT, Dynamic NAT, and PAT (NAT Overload) on a Cisco router. Compare how each method translates internal addresses.

## Environment

* **Tool:** Cisco Packet Tracer
* **Router:** R1
* **LAN:** `192.168.1.0/24`
* **R1 inside interface:** G0/0 — `192.168.1.1/24`
* **R1 outside interface:** G0/1 — `203.0.113.1/30`
* **ISP next hop:** `203.0.113.2`
* **Web server:** `198.51.100.10`
* **Dynamic NAT pool:** `203.0.113.10–203.0.113.13`

The `203.0.113.0/24` and `198.51.100.0/24` ranges are reserved for documentation. They are used here as simulated addresses in Packet Tracer, not as real public internet addresses.

## 1. Prepare NAT Interfaces

Mark the LAN-facing interface as inside and the ISP-facing interface as outside:

```text
configure terminal
interface gigabitEthernet 0/0
 ip nat inside
exit
interface gigabitEthernet 0/1
 ip nat outside
exit
end
```

These commands identify the inside and outside sides of NAT. They do not create a translation by themselves.

## 2. Static NAT

Create a permanent one-to-one mapping for PC1:

```text
configure terminal
ip nat inside source static 192.168.1.10 203.0.113.10
end
```

Test from PC1:

```text
ping 198.51.100.10
```

Verify on R1:

```text
show ip nat translations
```

PC1's inside local address (`192.168.1.10`) should map to the inside global address (`203.0.113.10`).

Remove the static mapping before continuing:

```text
configure terminal
no ip nat inside source static 192.168.1.10 203.0.113.10
end
```

## 3. Dynamic NAT

### Create an ACL

The ACL identifies the internal subnet whose traffic can be translated:

```text
configure terminal
access-list 1 permit 192.168.1.0 0.0.0.255
```

### Create the public address pool

```text
ip nat pool NAT-POOL 203.0.113.10 203.0.113.13 netmask 255.255.255.0
```

This pool contains four addresses that R1 can assign dynamically.

### Connect the ACL to the pool

```text
ip nat inside source list 1 pool NAT-POOL
end
```

Test from more than one PC by pinging `198.51.100.10`, then inspect the translations:

```text
show ip nat translations
```

When PC1 and PC2 had active translations at the same time, R1 assigned them different inside global addresses from the pool.

**Important:** Dynamic NAT uses a separate public IP for each active inside-host mapping. If all four pool addresses are occupied, another host cannot obtain a translation until an address becomes available.

### Remove Dynamic NAT

```text
configure terminal
no ip nat inside source list 1 pool NAT-POOL
no ip nat pool NAT-POOL
no access-list 1
end
```

## 4. PAT (NAT Overload)

PAT lets multiple internal devices share the IP address of R1's outside interface. It distinguishes their translations using transport ports; for ICMP, it uses ICMP identifiers.

Recreate the ACL:

```text
configure terminal
access-list 1 permit 192.168.1.0 0.0.0.255
```

Configure PAT using G0/1's address:

```text
ip nat inside source list 1 interface gigabitEthernet 0/1 overload
end
```

Test connectivity from PC1 and PC2, then run:

```text
show ip nat translations
```

Both PCs should have the same inside global address (`203.0.113.1`), but different translated identifiers for their ICMP traffic. In the output, the values after the colon for ICMP are identifiers, not TCP/UDP port numbers.

## 5. Remove NAT Configuration

Remove the PAT rule and ACL, then remove the inside/outside interface designations:

```text
configure terminal
no ip nat inside source list 1 interface gigabitEthernet 0/1 overload
no access-list 1
interface gigabitEthernet 0/0
 no ip nat inside
exit
interface gigabitEthernet 0/1
 no ip nat outside
exit
end
```

Verify that the NAT configuration is gone:

```text
show ip nat translations
show running-config
```

The NAT translations should be empty, and the running configuration should no longer contain NAT commands. The interface IP addresses and default route should remain configured.

## Key Commands

| Command | Purpose |
|---|---|
| `show ip nat translations` | View active NAT translations |
| `ip nat inside` | Mark an inside interface |
| `ip nat outside` | Mark an outside interface |
| `ip nat inside source static <inside-local> <inside-global>` | Configure Static NAT |
| `access-list 1 permit <network> <wildcard>` | Select inside addresses for translation |
| `ip nat pool <name> <start-ip> <end-ip> netmask <mask>` | Create a Dynamic NAT pool |
| `ip nat inside source list 1 pool <name>` | Configure Dynamic NAT |
| `ip nat inside source list 1 interface <outside-interface> overload` | Configure PAT |

## What I Learned

* Static NAT creates a permanent one-to-one mapping.
* Dynamic NAT assigns addresses from a pool; the pool limits the number of simultaneous inside-host mappings.
* PAT allows multiple internal devices to share one outside IP by distinguishing their connections.
* The NAT translation table shows the original and translated addresses, along with the outside destination.
