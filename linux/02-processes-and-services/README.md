# Linux Lab 02 — Processes & Services

## Objective

Practice investigating running processes, managing Linux services, identifying processes listening on network ports, terminating processes, and analyzing service logs.

The goal was to understand how Linux processes and services work together and how these tools can be used for troubleshooting.

## Environment

* **OS:** Ubuntu Linux
* **Environment:** Virtual Machine
* **Service:** Apache2
* **Tools:** `ps`, `top`, `systemctl`, `ss`, `kill`, `tail`

## Scenario

I was working as a junior Linux administrator investigating a Linux server running a web service.

The tasks were to:

* Inspect running processes
* Monitor resource usage
* Check running services
* Verify the Apache web server
* Start, stop, and restart a service
* Identify processes listening on network ports
* Find and terminate a process
* Investigate service logs
* Monitor logs in real time

---

## 1. Process Investigation

I first opened Firefox to create additional activity on the system.

I used:

```bash
ps aux
```

This displayed the running processes on the system, including processes belonging to different users.

I then used:

```bash
top
```

to monitor running processes and observe CPU and memory usage in real time.

### What I Learned

`ps aux` provides a snapshot of running processes, while `top` provides a continuously updating view of processes and resource usage.

---

## 2. Service Management

I used `systemctl` to inspect and manage system services.

I identified Apache2 as a loaded and running service.

I then checked its status with:

```bash
systemctl status apache2
```

This confirmed that Apache was active and running.

I also practiced controlling the service using:

```bash
systemctl stop apache2
systemctl start apache2
systemctl restart apache2
```

### What I Learned

`systemctl` is used to manage services controlled by `systemd`.

The main operations I practiced were:

* `start` — start a service
* `stop` — stop a service
* `restart` — restart a service
* `status` — inspect the current state of a service

---

## 3. Identifying Listening Ports and Processes

Initially, I did not know how to determine which process was listening on a TCP port.

I used:

```bash
sudo ss -tulpn
```

This displayed listening network sockets and the processes associated with them.

I found multiple Apache2 processes with different PIDs.

### What I Learned

A single service can create multiple processes.

For example:

```text
Apache2 service
      |
Multiple Apache2 processes
      |
Different PIDs
```

The PID (Process ID) uniquely identifies a particular running process.

This is useful when troubleshooting which application is using a specific network port.

---

## 4. Finding and Terminating a Process

I created a long-running `ping` process:

```bash
ping -c 100 cisco.com
```

I then used `Ctrl + Z`, which suspended the process.

After finding its PID using process information, I terminated it with:

```bash
kill -9 <PID>
```

I then verified that the process was no longer running.

### What I Learned

`Ctrl + Z` does not terminate a process. It suspends the foreground job.

`kill -9 <PID>` sends a forceful termination signal to the specified process.

---

## 5. Investigating Apache Logs

I explored the Linux log directory and found the Apache logs under:

```text
/var/log/apache2/
```

Apache provides logs such as:

* `access.log` — records requests received by the web server
* `error.log` — records errors and other diagnostic information

I used `tail` to inspect recent entries:

```bash
tail -n 20 /var/log/apache2/error.log
```

I also used:

```bash
tail -n 20 /var/log/apache2/access.log
```

to view recent web requests.

---

## 6. Monitoring Logs in Real Time

I used:

```bash
tail -f /var/log/apache2/access.log
```

The `-f` option continuously follows the file and displays new entries as they are added.

I generated activity on the Apache server and observed new entries appear in the terminal.

I stopped the live monitoring with:

```text
Ctrl + C
```

### What I Learned

`tail -f` is useful when troubleshooting services because it allows administrators to watch logs as events happen.

---

## Troubleshooting

One of the challenges during this lab was determining why Apache was not initially appearing when I inspected listening network ports.

I checked the Apache service status and then used:

```bash
sudo ss -tulpn
```

After confirming that Apache was running, I was able to identify multiple Apache processes and their PIDs.

Another important discovery was understanding that multiple Apache processes can belong to the same service.

---

## What I Learned

This lab helped me connect several Linux administration concepts:

* Processes are individual running instances of programs.
* Every process has a PID.
* Services can manage and create processes.
* `systemctl` can control services.
* `ss` can identify listening network sockets and associated processes.
* A single service can have multiple processes.
* `kill` can be used to terminate processes.
* `tail` can be used to inspect logs.
* `tail -f` can monitor logs in real time.
* Linux logs are valuable for troubleshooting.

```

 This lab showed me how different Linux tools can be combined instead of using each command independently.

