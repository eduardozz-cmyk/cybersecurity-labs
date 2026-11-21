# DNS and NAT Troubleshooting

## Scenario

The Admin laptop could not reach the Timecard server. I investigated the DNS and NAT configuration on the Packet Tracer network to find the cause.

## Environment

- **Tool:** Cisco Packet Tracer
- **Devices:** Admin laptop, Edge router, DNS server, Timecard server
- **Timecard server:** `215.1.1.49`
- **Edge inside interface:** G0/0/0 — `172.16.20.10/30`
- **Edge outside interface:** G0/0/1 — `215.1.1.50/30`

## Investigation

### DNS

I tested the hostname from the Admin laptop:

```text
ping timecard.com
```

The output showed that the name resolved to `214.1.1.10`, which did not match the Timecard server's address (`215.1.1.49`). I checked the DNS A record and found that it contained the incorrect address.

### NAT

I connected to the Edge router and inspected its configuration with:

```cisco
show running-config
```

The NAT configuration used this pool and translation rule:

```cisco
ip nat pool NAT_POOL 192.168.100.10 192.168.100.50 netmask 255.255.255.0
ip nat inside source list NAT_LIST pool NAT_POOL
```

The pool contained private addresses, even though the router was translating traffic toward the external network. The outside interface already had the address `215.1.1.50`.

## Changes Made

I corrected the DNS A record:

```text
timecard.com → 215.1.1.49
```

I then replaced the pool-based dynamic NAT rule with PAT. The router would use its outside interface address, while keeping the existing `NAT_LIST` ACL and inside/outside interface settings.

I removed the translation rule before deleting the pool:

```cisco
no ip nat inside source list NAT_LIST pool NAT_POOL
no ip nat pool NAT_POOL
```

The pool could not be deleted while the rule still referenced it, so removing the rule first resolved that dependency.

I configured PAT:

```cisco
ip nat inside source list NAT_LIST interface GigabitEthernet0/0/1 overload
```

## Verification

After both changes, I tested from the Admin laptop:

| Test | Result |
|---|---|
| `ping 215.1.1.49` | Successful |
| `ping timecard.com` | Successful |

Both tests worked. The DNS name resolved to the correct server address, and the Admin laptop could reach the server.

## What I Learned

- A hostname resolving to an IP does not prove that the destination is reachable.
- DNS resolution and NAT are separate parts of the connection and need to be checked independently.
- The ACL selects which inside addresses are eligible for translation; the NAT rule determines how they are translated.
- PAT lets multiple internal devices share the outside interface's IP address.
- A NAT pool cannot be removed while an active translation rule still references it.
