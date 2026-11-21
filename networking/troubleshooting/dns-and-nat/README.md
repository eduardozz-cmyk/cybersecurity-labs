# DNS and NAT Troubleshooting

## Objective

Investigate why the Admin laptop could not reach the Timecard server, identify the causes, and restore connectivity.

## Environment

- **Tool:** Cisco Packet Tracer
- **Devices:** Admin laptop, Edge router, DNS server, and Timecard server
- **DNS server:** `172.16.30.250`
- **Edge inside interface:** G0/0/0 — `172.16.20.10/30`
- **Edge outside interface:** G0/0/1 — `215.1.1.50/30`
- **Timecard server:** `215.1.1.49`

## 1. Investigate DNS Resolution

Start from the Admin laptop and test the server by its hostname:

```text
ping timecard.com
```

The ping failed, and the output showed that `timecard.com` resolved to `214.1.1.10`. That did not match the Timecard server's address in the topology (`215.1.1.49`).

Before changing DNS, check that the Admin laptop can reach its DNS server:

```text
ping 172.16.30.250
```

The DNS server responded, so it was reachable. In the DNS server's **Services → DNS** settings, inspect the A record and confirm that it maps:

```text
timecard.com → 214.1.1.10
```

The record contained the wrong address. Correct it to:

```text
timecard.com → 215.1.1.49
```

This fixes the name-to-address mapping. It does not, by itself, guarantee that packets can reach the server.

## 2. Inspect the Edge Router's NAT Configuration

The destination address was now identified correctly, so the next step was to inspect how the Edge router translated traffic leaving the internal network.

From the Admin laptop, connect to the Edge router using SSH:

```text
ssh -l admin 172.16.20.10
```

Then inspect its configuration:

```cisco
show running-config
```

The NAT section contained:

```cisco
ip nat pool NAT_POOL 192.168.100.10 192.168.100.50 netmask 255.255.255.0
ip nat inside source list NAT_LIST pool NAT_POOL
```

The router's outside interface was `GigabitEthernet0/0/1`, with address `215.1.1.50`. However, the dynamic NAT pool contained private addresses (`192.168.100.10–192.168.100.50`).

Those private addresses are not publicly routable. The router therefore needed to translate inside traffic to an address usable on its outside network.

## 3. Replace Dynamic NAT with PAT

Remove the NAT rule first, then delete the pool it references:

```cisco
no ip nat inside source list NAT_LIST pool NAT_POOL
no ip nat pool NAT_POOL
```

Keep the `NAT_LIST` access list and the existing inside/outside interface designations.

Configure PAT using the outside interface:

```cisco
ip nat inside source list NAT_LIST interface GigabitEthernet0/0/1 overload
```

This translates eligible inside addresses to the outside interface's address (`215.1.1.50`). The `overload` keyword allows multiple internal devices to share that address by distinguishing their sessions with port numbers (or protocol-specific identifiers, such as ICMP identifiers).

## 4. Verify the Fixes

From the Admin laptop, test the Timecard server directly by IP:

```text
ping 215.1.1.49
```

Then test using its hostname:

```text
ping timecard.com
```

Both tests succeeded. The hostname resolved to the corrected server address, and the server was reachable.

## Key Takeaways

- Check what address a hostname resolves to before assuming the destination server is unreachable.
- Verify that the DNS server itself is reachable, then inspect its A record.
- DNS provides the destination IP; NAT determines how eligible inside source addresses are translated.
- A dynamic NAT pool must contain addresses appropriate for the outside network.
- PAT can use the router's outside interface address to translate traffic from multiple internal devices.
- Remove a NAT rule before deleting a pool that the rule references.
