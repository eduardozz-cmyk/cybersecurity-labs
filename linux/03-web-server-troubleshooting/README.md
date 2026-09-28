# Linux Lab 03 — Web Server Troubleshooting

## Objective

Troubleshoot a Linux web server that is running but not accepting HTTP connections on the expected port.

The goal was to identify the cause of the problem, correct the Apache configuration, validate the change, and verify that the web server was functioning correctly.

## Environment

* **OS:** Ubuntu Linux
* **Environment:** Virtual Machine
* **Web Server:** Apache2
* **Tools:** `systemctl`, `ss`, `grep`, `ls`, `cat`, `nano`, `apache2ctl`, `curl`

## Scenario

I was investigating an Ubuntu server hosting an internal website.

The reported problem was that the website was unreachable, even though Apache was expected to be running.

The investigation focused on determining whether:

* Apache was running
* Apache was listening on the expected network port
* The Apache configuration contained an incorrect port
* The configuration could be safely changed
* The web server responded after the fix

## 1. Check Apache Status

I first checked whether the Apache service was running:

```bash
systemctl status apache2
```

Apache was reported as:

```text
active (running)
```

This showed that the problem was not caused by Apache being stopped.

## 2. Check Listening Ports

I then checked the network sockets and processes:

```bash
sudo ss -tulpn
```

Apache was not listening on the expected HTTP port `80` or HTTPS port `443`.

This indicated that Apache was running but was not listening on the expected HTTP port.

## 3. Investigate Apache Configuration

I first searched the main Apache port configuration:

```bash
grep "Listen" /etc/apache2/ports.conf
```

This did not return any results.

I then inspected the Apache configuration directory:

```bash
ls /etc/apache2/
```

After confirming that `ports.conf` existed, I displayed its contents:

```bash
cat /etc/apache2/ports.conf
```

The configuration contained:

```text
Listen 81
Listen 443
Listen 443
```

The important finding was that Apache was configured to listen for HTTP traffic on port `81` instead of the standard HTTP port `80`.

## 4. Confirm the Configuration Problem

I verified the active listening ports again:

```bash
sudo ss -tulpn
```

Apache was listening on port `81`.

This confirmed that the configuration was responsible for the problem.

The expected configuration was:

```text
Listen 80
```

## 5. Correct the Configuration

I edited the Apache port configuration:

```bash
sudo nano /etc/apache2/ports.conf
```

I changed:

```text
Listen 81
```

to:

```text
Listen 80
```

## 6. Validate the Configuration

Before restarting Apache, I checked the configuration for syntax errors:

```bash
sudo apache2ctl configtest
```

The result was:

```text
Syntax OK
```

This confirmed that the configuration was valid and could be safely applied.

## 7. Restart Apache

I restarted the Apache service:

```bash
sudo systemctl restart apache2
```

Then verified its status:

```bash
sudo systemctl status apache2
```

Apache was successfully running:

```text
active (running)
```

## 8. Verify the Listening Port

I checked the listening ports again:

```bash
sudo ss -tulpn
```

Apache was now listening on:

```text
:80
```

This confirmed that the configuration change had been applied successfully.

## 9. Test the Web Server

Finally, I tested the web server locally using:

```bash
curl http://localhost
```

The command returned the HTML content of the webpage.

This confirmed that Apache was not only running and listening on port 80, but was also successfully responding to HTTP requests.

## Troubleshooting Process

```text
Website unreachable
        |
Check Apache status
        |
Apache active
        |
Check listening ports
        |
Port 80 not in use
        |
Inspect Apache configuration
        |
Found Listen 81
        |
Change 81 to 80
        |
Run configtest
        |
Syntax OK
        |
Restart Apache
        |
Confirm port 80
        |
Test with curl
        |
Website responding
```

## What I Learned

* A service can be running while still being incorrectly configured.
* `systemctl status` can determine whether a service is active.
* `ss -tulpn` can identify listening ports and associated processes.
* Apache's listening ports can be configured in `/etc/apache2/ports.conf`.
* `grep` can search configuration files for specific settings.
* `apache2ctl configtest` can validate Apache configuration before restarting the service.
* `curl` can be used to test a web server directly from the command line.
* Troubleshooting should verify actual service behavior instead of assuming that an `active (running)` status means everything is working.

