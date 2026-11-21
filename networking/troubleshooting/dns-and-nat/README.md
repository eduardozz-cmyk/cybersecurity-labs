# DNS and NAT Troubleshooting

## Objective

Investigate why Hosts A and C cannot access the cloud-hosted timecard system at `timecards.com`. Check network connectivity, identify the DNS and NAT issues, apply the necessary corrections, and verify that the service is reachable.

## Environment

- **Tool:** Cisco Packet Tracer
- **Hosts:** Hosts A, B, and C; Admin laptop
- **DNS server:** `172.16.30.250`
- **Timecard server:** `215.1.1.49`
- **Edge router:** Internal interface `172.16.20.10`; external interface `215.1.1.50`
- **Routing and translation:** Existing routed network with NAT on the Edge router

## Scenario

Several new PCs were recently connected to the network. Users at Hosts A and C reported that they could not access the cloud-hosted timecard system using `timecards.com`.

DNS was suggested as a possible cause. I started by checking the hosts' network configuration and connectivity before investigating DNS, routing, and NAT.

## 1. Initial connectivity checks

I started on Host A by running:

```cmd
ipconfig
```

I used the output to identify the host's IP configuration and default gateway. Host A was configured with the address `10.10.10.5/24`, and its default gateway was `10.10.10.1`.

I first checked whether the default gateway was reachable:

```cmd
ping 10.10.10.1
```

The ping succeeded. I then checked connectivity to the DNS server:

```cmd
ping 172.16.30.250
```

This also succeeded.

Next, I tested the timecard server directly by IP address:

```cmd
ping 215.1.1.49
```

The request timed out. Since the server could not be reached by IP address, the issue was not just DNS name resolution.

I also tested the Core router's address on the Edge–Core transit network:

```cmd
ping 172.16.20.9
```

This test failed, giving me another point to investigate along the network path.

## 2. Tracing the route

From Host C, I ran:

```cmd
tracert 215.1.1.49
```

The trace showed these Layer 3 hops:

```text
1  10.30.30.3
2  172.16.20.5
3  172.16.20.10
4  Request timed out
```

The traffic reached the Edge router's internal interface at `172.16.20.10`, but the trace did not reach the destination. This helped narrow the investigation towards the Edge router and its external connectivity.

## 3. Checking DNS

I inspected the DNS service and found that the A record for `timecards.com` pointed to the wrong IP address:

```text
timecards.com → 214.1.1.10
```

The timecard server's actual address was `215.1.1.49`, so I corrected the record:

```text
timecards.com → 215.1.1.49
```

This corrected the hostname's destination, but the server was still unreachable by IP address. I continued investigating rather than treating DNS as the only problem.

## 4. Inspecting and correcting NAT

To investigate the Edge router, I connected to it remotely using SSH. Once connected, I inspected its running configuration:

```cisco
show running-config
```

The Edge router had its internal interface configured as NAT inside and its external interface as NAT outside:

- **G0/0/0:** `172.16.20.10` — `ip nat inside`
- **G0/0/1:** `215.1.1.50` — `ip nat outside`

The existing dynamic NAT configuration used this pool:

```cisco
ip nat pool NAT_POOL 192.168.100.10 192.168.100.50 netmask 255.255.255.0
ip nat inside source list NAT_LIST pool NAT_POOL
```

The pool used private addresses, which were not suitable as translated source addresses for communication over the external network. The external network was `215.1.1.48/30`, so I used the Edge router's outside interface address for PAT instead.

I first removed the NAT rule that referenced the pool:

```cisco
no ip nat inside source list NAT_LIST pool NAT_POOL
```

Then I deleted the pool:

```cisco
no ip nat pool NAT_POOL
```

The router initially rejected the pool deletion because it was still referenced by the NAT rule. Removing the rule first allowed me to delete the pool.

I kept the existing `NAT_LIST` access list and the NAT inside/outside interface settings, then configured PAT:

```cisco
ip nat inside source list NAT_LIST interface GigabitEthernet0/0/1 overload
```

## 5. Testing and results

After correcting the DNS record and replacing the dynamic NAT pool with PAT, I tested connectivity from the Admin laptop:

```cmd
ping 215.1.1.49
ping timecards.com
```

Both tests succeeded. The hostname resolved to `215.1.1.49`, and the server responded to the ping.

## Key concepts and lessons learned

- I started with `ipconfig` and basic connectivity checks instead of assuming DNS was the only cause.
- A host can reach its default gateway and DNS server while still being unable to reach an external destination.
- Testing a destination by IP address helps distinguish a connectivity problem from a name-resolution problem.
- Traceroute helped me see how far traffic travelled through the Layer 3 path.
- SSH allowed me to access the Edge router remotely and inspect its configuration.
- DNS resolves a hostname to an IP address; NAT translates source addresses. Fixing one does not automatically fix the other.
- The original NAT pool used private addresses. PAT allowed internal hosts to share the Edge router's outside interface address, using port numbers to distinguish their connections.
- A NAT pool must no longer be referenced by a NAT rule before it can be removed.
