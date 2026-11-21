# DNS and NAT Troubleshooting

## 1. Objective and environment

Several new PCs had recently been connected to the network. Users at Hosts A and C could not access the cloud-hosted timecard system at `timecards.com`.

The objective was to investigate the connectivity problem, identify the cause and restore access. Although DNS was suggested as a possible cause, I started by checking basic network connectivity before focusing on DNS.

**Environment:** Cisco Packet Tracer, multiple LANs connected through routers and switches, a DNS server, and an Edge router providing external connectivity and NAT.

## 2. Implementation and configuration

The network already had routing, DNS and NAT configured. During the investigation, I inspected the DNS A record and the Edge router's NAT configuration.

The relevant original NAT configuration was:

```cisco
ip nat pool NAT_POOL 192.168.100.10 192.168.100.50 netmask 255.255.255.0
ip nat inside source list NAT_LIST pool NAT_POOL
```

The Edge router used G0/0/0 (`172.16.20.10`) as its NAT inside interface and G0/0/1 (`215.1.1.50`) as its NAT outside interface.

## 3. Troubleshooting process

### Initial checks

I started on Host A by checking its IP configuration:

```cmd
ipconfig
```

This showed the host's network settings, including its default gateway. I used the gateway address to begin checking local connectivity.

I tested the default gateway:

```cmd
ping 10.10.10.1
```

The gateway responded. I then tested the DNS server:

```cmd
ping 172.16.30.250
```

This also succeeded, confirming that Host A could reach its gateway and the DNS server.

However, a direct ping to the Timecard server failed:

```cmd
ping 215.1.1.49
```

This showed that the problem was not limited to hostname resolution.

### Tracing the path

I used `tracert` from Host C to investigate how far traffic travelled toward the server:

```cmd
tracert 215.1.1.49
```

The trace reached the Edge router's internal side before timing out. I also checked the Edge router's external interface (`215.1.1.50`), which was reachable, while the Timecard server (`215.1.1.49`) was not.

These checks shifted the investigation toward the Edge router's external connectivity and NAT configuration.

### Checking DNS

I inspected the DNS service and found that the A record for the timecard system pointed to the wrong address:

```text
timecards.com → 214.1.1.10
```

I corrected it to the server's address:

```text
timecards.com → 215.1.1.49
```

Correcting the record addressed the name-resolution problem, but the server was still unreachable by IP address. This indicated that DNS was not the only issue.

### Inspecting NAT

I reviewed the Edge router's running configuration:

```cisco
show running-config
```

The router had a default route through its external interface and the expected NAT inside/outside interface designations.

However, the dynamic NAT pool used private addresses (`192.168.100.10–192.168.100.50`). These were not suitable as Internet-facing translated addresses in this topology. The outside network was `215.1.1.48/30`, so the router's outside interface address (`215.1.1.50`) was the address available to use for PAT.

## 4. Testing and results

I replaced the pool-based dynamic NAT rule with PAT.

First, I removed the NAT rule that referenced the pool, then deleted the pool. The router initially rejected the pool deletion because the pool was still in use.

```cisco
no ip nat inside source list NAT_LIST pool NAT_POOL
no ip nat pool NAT_POOL
```

I kept the existing `NAT_LIST` ACL and the NAT inside/outside interface settings, then configured PAT using the outside interface:

```cisco
ip nat inside source list NAT_LIST interface GigabitEthernet0/0/1 overload
```

After correcting DNS and replacing dynamic NAT with PAT, I tested from the Admin laptop:

| Test | Result |
|---|---|
| `ping 215.1.1.49` | Successful |
| `ping timecards.com` | Resolved to `215.1.1.49` and successful |

Both tests succeeded, confirming that the server was reachable by IP address and hostname.

## 5. Key concepts and lessons learned

- Start troubleshooting with the host's IP configuration and basic connectivity checks before assuming the suggested cause is correct.
- A successful ping to the default gateway and DNS server does not guarantee connectivity to an external destination.
- Traceroute helps show how far traffic gets through the Layer 3 path.
- DNS resolves a hostname to an IP address; NAT translates eligible source addresses. They solve different problems.
- Dynamic NAT requires a suitable pool of externally routable addresses. PAT can let multiple internal devices share the router's outside interface address by distinguishing sessions with port numbers.
- A NAT pool cannot be deleted while a NAT rule still references it.
