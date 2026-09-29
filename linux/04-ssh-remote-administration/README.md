# Linux Lab 04 — SSH & Remote Administration

## Objective

Learn how to remotely access and administer an Ubuntu Linux machine using SSH.

The goal was to configure an SSH server, troubleshoot the service, verify that it was listening on the correct port, establish a remote connection from Windows, perform administrative tasks through the SSH session, and safely disconnect.

## Environment

* **Client:** Windows
* **Server:** Ubuntu Linux 26.04.1 LTS
* **Environment:** VirtualBox
* **Network Mode:** NAT
* **SSH:** OpenSSH Server
* **Tools:** `systemctl`, `dpkg`, `apt`, `ss`, `whoami`, `ip`, `hostname`, `pwd`, `ls`, `touch`, `exit`

## Scenario

I needed to administer an Ubuntu virtual machine remotely from my Windows host.

The Ubuntu VM was using VirtualBox NAT networking, and SSH was not initially installed.

The investigation focused on:

* Identifying the Ubuntu server's IP address
* Checking whether SSH was available
* Installing the SSH server
* Starting and enabling the SSH service
* Verifying that SSH was listening on TCP port 22
* Configuring VirtualBox NAT port forwarding
* Connecting from Windows using SSH
* Performing a remote administration task
* Verifying the changes from the Ubuntu terminal
* Disconnecting safely

## 1. Identify the Ubuntu IP Address

I first checked the network interfaces:

```bash
ip addr
```

The Ubuntu VM had the following IP address:

```text
10.0.2.15/24
```

The `/24` represents the network prefix.

The address `10.0.2.255` was identified as the broadcast address rather than the machine's IP address.

## 2. Check the SSH Service

I checked whether the SSH service existed:

```bash
systemctl status ssh
```

The system reported that the SSH service could not be found.

This indicated that the SSH server was not installed.

## 3. Check Whether OpenSSH Server Was Installed

I used `dpkg` to check the package:

```bash
dpkg -l openssh-server
```

The package showed the `un` status, indicating that `openssh-server` was not installed.

## 4. Install OpenSSH Server

I installed the OpenSSH server package:

```bash
sudo apt install openssh-server
```

After installation, I checked the SSH service again:

```bash
systemctl status ssh
```

The service existed but was initially inactive.

## 5. Start and Enable SSH

I started the SSH service and configured it to start automatically when Ubuntu boots:

```bash
sudo systemctl start ssh
sudo systemctl enable ssh
```

I also used:

```bash
sudo systemctl restart ssh
```

to restart the service.

Finally, I checked its status:

```bash
systemctl status ssh
```

The SSH service was now:

```text
active (running)
```

## 6. Verify SSH Listening Port

I used the network socket information learned in a previous lab:

```bash
sudo ss -tulpn
```

SSH was listening on:

```text
[::]:22
```

This confirmed that the SSH server was listening on its default TCP port, **22**.

## 7. Configure VirtualBox NAT Port Forwarding

The Ubuntu VM was using VirtualBox NAT networking.

With NAT, the Windows host could not directly connect to the VM using its `10.0.2.15` address.

I configured a VirtualBox NAT port forwarding rule:

| Setting    | Value     |
| ---------- | --------- |
| Name       | SSH       |
| Protocol   | TCP       |
| Host IP    | Blank     |
| Host Port  | 2222      |
| Guest IP   | 10.0.2.15 |
| Guest Port | 22        |

This created the following path:

```text
Windows localhost:2222
        |
        | VirtualBox NAT
        ↓
Ubuntu 10.0.2.15:22
```

The host uses port `2222`, while SSH continues using its normal port `22` inside Ubuntu.

## 8. Connect to Ubuntu from Windows

From Windows PowerShell, I connected using:

```powershell
ssh vboxuser@localhost -p 2222
```

The first connection displayed a host authenticity warning and asked whether the host should be added to the list of known hosts.

After accepting it and entering the Ubuntu user's password, the SSH connection was established successfully.

The Ubuntu login message confirmed that I was now connected to the Ubuntu system remotely.

## 9. Verify the Remote Session

Inside the SSH session, I used several Linux commands:

```bash
whoami
```

This confirmed that I was connected as:

```text
vboxuser
```

I also used:

```bash
hostname
```

to identify the remote machine.

I used:

```bash
ip addr
```

to verify the Ubuntu network configuration.

The server's IP address was:

```text
10.0.2.15/24
```

## 10. Perform Remote Administration

While connected through SSH from Windows, I created a file:

```bash
touch ssh-test.txt
```

I then verified it:

```bash
ls
```

The file appeared in the Ubuntu user's home directory.

I opened Ubuntu's normal local terminal and ran:

```bash
ls
```

The same `ssh-test.txt` file was visible.

This confirmed that the SSH session was interacting with the same Ubuntu filesystem as the local terminal.

## 11. Disconnect

After completing the remote administration task, I disconnected from the SSH session using:

```bash
exit
```

This returned me to the normal Windows PowerShell session.

## Troubleshooting Process

```text
Need remote access to Ubuntu
        |
Find Ubuntu IP
        |
10.0.2.15
        |
Check SSH service
        |
Service not found
        |
Check package
        |
openssh-server not installed
        |
Install OpenSSH Server
        |
SSH service inactive
        |
Start and enable SSH
        |
Verify service
        |
Active (running)
        |
Check listening ports
        |
SSH listening on port 22
        |
Check VirtualBox network
        |
NAT
        |
Configure port forwarding
        |
Windows :2222 → Ubuntu :22
        |
Connect using SSH
        |
Remote administration successful
        |
Disconnect with exit
```

## What I Learned

* SSH provides encrypted remote access to Linux systems.
* The SSH client initiates the connection, while the SSH server accepts it.
* OpenSSH Server provides the SSH server functionality on Ubuntu.
* SSH normally uses TCP port `22`.
* `systemctl` can be used to check and manage the SSH service.
* `dpkg` can be used to determine whether a package is installed.
* `apt` can install required packages.
* `ss` can verify that a service is listening on a network port.
* VirtualBox NAT can require port forwarding when connecting from the host to a virtual machine.
* SSH allows commands and administrative tasks to be performed remotely.
* `exit` safely terminates an SSH session.

## Key Takeaway

A service being unavailable does not necessarily mean the configuration is wrong. The problem must be investigated step by step.

In this lab, SSH was initially unavailable because the OpenSSH server package was not installed. After installing and starting the service, I verified that it was listening on port 22.

Because the Ubuntu VM was using VirtualBox NAT networking, I then configured port forwarding from the Windows host's port `2222` to the Ubuntu SSH port `22`.

This allowed me to establish an SSH connection from Windows and perform administrative tasks on Ubuntu remotely.
