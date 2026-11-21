# DNS and NAT Troubleshooting

## Scenario

Several new PCs had recently been connected to the network. Users at Hosts A and C could not access the cloud-hosted timecard system at `timecards.com`. DNS was suggested as a possible cause, but I started by checking connectivity across the network rather than assuming DNS was the only problem.

## Investigation

### 1. Check local connectivity

I started with Host A and checked whether it could reach its default gateway:

```text
ping 10.10.10.1
```

The gateway responded. I then checked connectivity to the DNS server:

```text
ping 172.16.30.250
```

This also succeeded, confirming that Host A could reach its local gateway and the DNS server.

However, a direct ping to the Timecard server failed:

```text
ping 215.1.1.49
```

Host B also failed to reach the server by IP. This showed that the issue was not limited to hostname resolution.

### 2. Trace the path toward the server

I checked the path from Host C:

```text
tracert 215.1.1.49
```

The trace reached these Layer 3 hops before timing out:

```text
10.30.30.3
172.16.20.5
172.16.20.10
Request timed out
```

The packet reached the Edge router's internal interface (`172.16.20.10`), but did not reach the external server. I also confirmed that the Edge router's external interface (`215.1.1.50`) was reachable, while the Timecard server (`215.1.1.49`) was not.

These checks shifted the investigation toward the Edge router and the path beyond it.

### 3. Check DNS

I tested the hostname from the Admin laptop. It resolved to `214.1.1.10`, not the Timecard server's expected address, `215.1.1.49`.

The DNS server itself was reachable, so I inspected its DNS service and found that the A record for `timecard.com` pointed to the wrong address:

```text
timecard.com → 214.1.1.10
```

I corrected it to:

```text
timecard.com → 215.1.1.49
```

This fixed the name resolution, but direct connectivity to `215.1.1.49` was still failing. There was another issue to find.

### 4. Inspect the Edge router

I connected to the Edge router and reviewed its running configuration:

```cisco
show running-config
```

The router had a default route through its external interface and the expected inside/outside NAT interface designations:

- G0/0/0 — `172.16.20.10`, configured as `ip nat inside`
- G0/0/1 — `215.1.1.50`, configured as `ip nat outside`

However, its dynamic NAT configuration used this pool:

```cisco
ip nat pool NAT_POOL 192.168.100.10 192.168.100.50 netmask 255.255.255.0
ip nat inside source list NAT_LIST pool NAT_POOL
```

The pool contained private addresses, which are not routable on the external network. The outside network was `215.1.1.48/30`, so there were no additional usable public addresses in that subnet for a separate pool. The router's own outside address, `215.1.1.50`, could instead be used with PAT.

## Changes Made

I replaced the pool-based dynamic NAT rule with PAT.

First, I removed the rule that referenced the pool, then deleted the pool. The router initially rejected the pool deletion because it was still in use.

```cisco
no ip nat inside source list NAT_LIST pool NAT_POOL
no ip nat pool NAT_POOL
```

I kept the existing `NAT_LIST` ACL and the inside/outside interface settings, then configured PAT using the outside interface:

```cisco
ip nat inside source list NAT_LIST interface GigabitEthernet0/0/1 overload
```

## Verification

After correcting DNS and replacing dynamic NAT with PAT, I tested from the Admin laptop:

| Test | Result |
|---|---|
| `ping 215.1.1.49` | Successful |
| `ping timecard.com` | Resolved to `215.1.1.49` and successful |

Both tests succeeded, confirming that the server was reachable by IP address and hostname.

## What I Learned

- Troubleshooting should begin with basic connectivity checks before assuming the suggested cause is correct.
- A successful ping to the gateway and DNS server does not guarantee connectivity to an external destination.
- Traceroute helps identify how far traffic gets through the Layer 3 path.
- DNS resolution and NAT are separate: DNS resolves the destination name, while NAT translates eligible source addresses.
- Dynamic NAT needs a suitable pool of externally routable addresses; PAT can share the router's outside interface address.
- A NAT pool cannot be deleted while a translation rule still references it.
