# DNS and NAT Troubleshooting

## Objective

Restore access to the Timecard server by identifying and correcting two configuration problems: an incorrect DNS record and an unsuitable dynamic NAT configuration.

## Environment

- **Tool:** Cisco Packet Tracer
- **Devices:** Admin laptop, Edge router, DNS server, and Timecard server
- **Edge inside interface:** G0/0/0 — `172.16.20.10/30`
- **Edge outside interface:** G0/0/1 — `215.1.1.50/30`
- **Timecard server:** `215.1.1.49`

## Problems Found

The Admin laptop could not access the Timecard server by its hostname.

Two separate issues were identified:

1. The DNS record for `timecard.com` pointed to `214.1.1.10` instead of the server's actual address, `215.1.1.49`.
2. The Edge router used a dynamic NAT pool containing private addresses (`192.168.100.10–192.168.100.50`) for traffic going toward the external network.

## 1. Correct the DNS Record

Update the `timecard.com` A record so it points to the Timecard server:

```text
timecard.com → 215.1.1.49
```

DNS translates the hostname into the server's IP address. It does not provide network connectivity by itself.

## 2. Replace Dynamic NAT with PAT

The Edge router's NAT rule referenced the private-address pool:

```text
ip nat inside source list NAT_LIST pool NAT_POOL
```

That pool was unsuitable for this external connection because its addresses were private and not publicly routable.

Remove the rule before deleting the pool, since the rule depends on it:

```cisco
no ip nat inside source list NAT_LIST pool NAT_POOL
no ip nat pool NAT_POOL
```

Keep the existing `NAT_LIST` access list and the inside/outside interface designations. Replace the old rule with PAT using the outside interface:

```cisco
ip nat inside source list NAT_LIST interface GigabitEthernet0/0/1 overload
```

This uses the outside interface address (`215.1.1.50`) for translations. The `overload` keyword allows multiple internal devices to share that address by distinguishing their sessions with port numbers (or protocol-specific identifiers, such as ICMP identifiers).

## 3. Verify Connectivity

From the Admin laptop, test the server by IP address:

```text
ping 215.1.1.49
```

Then test by hostname:

```text
ping timecard.com
```

Both tests succeeded after the DNS record and NAT configuration were corrected.

## Key Takeaways

- DNS resolution and NAT are separate parts of a connection; fixing one does not fix the other.
- An ACL selects which inside addresses are eligible for translation.
- Dynamic NAT uses addresses from a configured pool. The pool must contain addresses appropriate for the external network.
- PAT can translate multiple internal devices using the router's outside interface address.
- Remove a NAT rule before deleting a pool that the rule references.
