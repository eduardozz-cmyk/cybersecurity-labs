# Linux Lab 05 — Network Troubleshooting

## Objective

Practice a systematic approach to troubleshooting network connectivity on a Linux system.

The goal was to determine whether a server had a working network configuration, Internet connectivity, DNS resolution, and HTTP/HTTPS connectivity.

## Environment

* **OS:** Ubuntu Linux 26.04.1 LTS
* **Environment:** Virtual Machine
* **Network:** VirtualBox NAT
* **Tools:** `ip`, `ping`, `nslookup`, `curl`

## Scenario

A user reported:

> "I can't access a website from this server."

The task was to investigate the problem step by step instead of immediately changing the system configuration.

The investigation focused on determining whether the problem was related to:

* The local network configuration
* Internet connectivity
* DNS resolution
* The default gateway
* HTTP/HTTPS connectivity

## 1. Check Network Configuration

I first inspected the network interfaces and IP addresses:

```bash
ip addr
```

The system showed:

* `127.0.0.1` — loopback address
* `10.0.2.15/24` — Ubuntu VM's IPv4 address
* A MAC address associated with the network interface

The presence of `10.0.2.15/24` confirmed that the Ubuntu VM had an IPv4 address assigned.

## 2. Test Internet Connectivity

Next, I tested connectivity to an external IP address:

```bash
ping -c 4 8.8.8.8
```

The command returned responses similar to:

```text
64 bytes from 8.8.8.8
```

This confirmed that the Ubuntu VM could communicate with an external IP address.

At this point, a complete loss of Internet connectivity could be ruled out.

## 3. Test DNS Resolution

Since the reported problem involved accessing a website by domain name, I tested DNS resolution:

```bash
nslookup cisco.com
```

The command returned a successful DNS response.

This confirmed that Ubuntu could resolve the domain name to an IP address.

Therefore, DNS was functioning correctly.

## 4. Test HTTPS Connectivity

I then tested communication with the website itself using:

```bash
curl https://www.cisco.com
```

The command returned a large amount of webpage content.

This confirmed that Ubuntu was able to establish an HTTPS connection and receive a response from the web server.

## 5. Check the Default Route

Finally, I inspected the routing table:

```bash
ip route
```

The output contained a route beginning with:

```text
default via ...
```

The default route tells Linux where to send traffic when there is no more specific route for the destination.

The gateway acts as the exit point from the local network toward other networks and the Internet.

For example:

```text
Ubuntu
10.0.2.15
    |
    v
Default Gateway
10.0.2.2
    |
    v
Internet
```

## Troubleshooting Process

```text
User cannot access a website
            |
            v
      Check IP address
            |
            v
      IP address exists
            |
            v
     Test external IP
            |
            v
       8.8.8.8 responds
            |
            v
       Test DNS resolution
            |
            v
       DNS works
            |
            v
       Test HTTPS with curl
            |
            v
       Website responds
            |
            v
       Check default route
            |
            v
       Default route exists
```

## Results

The investigation did not identify an actual network failure.

The Ubuntu VM had:

* A valid IPv4 address
* External network connectivity
* Working DNS resolution
* A default route
* Successful HTTPS connectivity

The original report could therefore not be reproduced during the investigation.

## What I Learned

* `ip addr` can be used to inspect network interfaces and IP addresses.
* `ping` can test basic IP connectivity.
* `nslookup` can test DNS resolution.
* `curl` can test application-layer HTTP/HTTPS connectivity.
* `ip route` can display the Linux routing table.
* The default gateway provides a route to destinations outside the local network.
* Different network problems can occur at different layers.
* A systematic troubleshooting process is more effective than randomly changing configurations.

## Key Takeaway

Network troubleshooting should be performed **layer by layer**.

Instead of assuming that a website problem is caused by DNS, the web server, or the network, each part of the connection should be tested independently.

In this lab, the network configuration, connectivity, DNS, routing, and HTTPS connection were all verified successfully.
