# Linux Lab 02 — Processes & Services

## 🎯 Objective

Practice investigating running processes, managing Linux services, identifying processes listening on network ports, terminating processes, and analyzing service logs.

The goal was to understand how Linux processes and services work together and how these tools can be used for troubleshooting.

## 🖥️ Environment

* **OS:** Ubuntu Linux
* **Environment:** Virtual Machine
* **Service:** Apache2
* **Tools:** `ps`, `top`, `systemctl`, `ss`, `kill`, `tail`

## 🧪 Scenario

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

### What I learned

`ps aux` provides a snapshot of running processes, while `top` provides a continuously updating view of processes and resource usage.

---

## 2. Service Management

I used `systemctl` to inspect and manage system services.

I identified **Apache2** as a loaded and running service.

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

### What I learned

`systemctl` is used to manage services controlled by `systemd`.

The main operations I practiced were:

* `start` — start a service
* `stop` — stop a service
* `restart` — restart a service
* `status` — inspect the current state of a service

---

## 3.
