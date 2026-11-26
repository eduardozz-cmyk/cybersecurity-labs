# OSPF Routing Loop

## Objective

Troubleshoot why users cannot access `networks.com` after their PCs were configured to use Google's DNS server (`8.8.8.8`). Identify the cause of the connectivity failure and restore the correct routing path while respecting the lab constraints.

## Environment

- **Simulation:** Cisco Packet Tracer
- **Routing protocol:** OSPF, Area 0
- **DNS server:** `8.8.8.8`
- **Web server:** `172.16.0.4`
- **Devices investigated:** PC0, SW1, RTR1, RTR2, RTR3 and SW5

## Scenario

Users reported that they could no longer access `networks.com`. Their PCs had recently been changed to use Google's DNS server instead of the local DNS server.

The investigation focused on Layer 3 connectivity because the DNS server's IP address was unreachable.

## Troubleshooting Process

### 1. Initial connectivity tests

Tested connectivity from the PCs to determine whether the issue was related to the DNS server or general network connectivity.

- PC0 could reach its default gateway.
- The Admin PC could reach the web server (`172.16.0.4`).
- Both PC0 and the Admin PC failed to reach the DNS server (`8.8.8.8`).

This indicated that basic local connectivity was working, but connectivity to the DNS network was failing.

### 2. Traceroute investigation

Ran the following command from the Admin PC:

```text
tracert 8.8.8.8
```

The output revealed that traffic was repeatedly alternating between:

- RTR1: `10.1.1.1`
- RTR3: `192.168.200.2`

This identified a routing loop between the two routers.

### 3. Investigating RTR1's routing table

Checked RTR1's routing table:

```cisco
show ip route
```

Found a static route to the DNS network:

```cisco
S 8.8.8.0/28 [1/0] via 192.168.200.2
```

RTR1 was forwarding DNS traffic to RTR3 through this static route.

### 4. Investigating RTR3's routing table

Checked RTR3's route to the DNS network:

```cisco
show ip route 8.8.8.8
```

RTR3 had learned the following route through OSPF:

```text
O 8.8.8.0/28 [110/4] via 192.168.200.1
```

This meant RTR3 was also forwarding traffic toward RTR1, creating the routing loop.

Further investigation revealed that RTR3's interface toward SW5 was administratively down. This prevented RTR3 from learning the DNS network directly from SW5 through that connection.

The lab prohibited changing the administrative status of interfaces, so enabling the interface was not an option.

### 5. Understanding the OSPF route

SW5 was advertising the DNS network through OSPF. RTR1 had learned this route and could reach the DNS network through RTR2.

However, RTR1 was not selecting the OSPF route because its static route had a lower administrative distance.

The two routes were:

| Route source | Administrative distance |
|---|---:|
| Static | 1 |
| OSPF | 110 |

Since lower administrative distance is preferred, RTR1 selected the static route even though it caused the routing loop.

### 6. Modifying the static route

The lab prohibited adding or removing static routes but allowed modifying them.

Changed the administrative distance of the existing static route on RTR1 from `1` to `120`:

```cisco
ip route 8.8.8.0 255.255.255.240 192.168.200.2 120
```

This made the OSPF route more preferable because its administrative distance was lower.

### 7. Refreshing the routing table

Cleared RTR1's routing table to trigger route recalculation:

```cisco
clear ip route *
```

Then checked the route again:

```cisco
show ip route 8.8.8.8
```

The output confirmed that RTR1 was now using OSPF:

```text
Known via "ospf 1", distance 110, metric 3
Last update from 192.168.100.2
```

The next hop, `192.168.100.2`, was RTR2.

## Testing and Results

- Confirmed that the routing loop existed using traceroute.
- Identified the incorrect static route on RTR1.
- Modified its administrative distance from `1` to `120`.
- Cleared the routing table to trigger recalculation.
- Verified that RTR1 selected the OSPF route to `8.8.8.0/28` through RTR2.
- Successfully pinged the DNS server (`8.8.8.8`) from PC0.
- Successfully accessed `networks.com` after the routing change.

## Key Concepts and Lessons Learned

- **OSPF:** A dynamic routing protocol that allows routers to learn remote networks and calculate routes automatically.
- **LSAs and LSDB:** OSPF devices exchange link-state advertisements and maintain a link-state database describing the network topology.
- **Administrative distance:** Determines which routing source is preferred when multiple sources provide a route to the same destination. Lower values are preferred.
- **OSPF metric:** Determines the preferred path among routes learned through OSPF. Lower cost is generally preferred.
- **Routing loops:** Occur when routers repeatedly forward traffic toward each other because of incorrect routing information or route selection.
- **Static and dynamic routing:** Static routes can coexist with OSPF, but an incorrectly preferred static route can override a valid dynamic route.

The main lesson was that OSPF was functioning, but RTR1 preferred an incorrect static route because its administrative distance was lower than OSPF's. Increasing the static route's administrative distance allowed RTR1 to use the correct OSPF path instead.
