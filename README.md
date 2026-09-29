
# Apache HTTP Server Setup Lab

![Ubuntu](https://img.shields.io/badge/OS-Ubuntu%20Linux-E95420?logo=ubuntu&logoColor=white)
![Apache](https://img.shields.io/badge/Web%20Server-Apache2-D22128?logo=apache&logoColor=white)
![Focus](https://img.shields.io/badge/Focus-Linux%20Fundamentals-blue)

A hands-on lab covering the installation, service management, and virtual host configuration of the **Apache HTTP Server** on an Ubuntu Linux virtual machine.

---

## Table of Contents

- [Overview](#overview)
- [Skills Demonstrated](#skills-demonstrated)
- [Prerequisites](#prerequisites)
- [Step 1: Installing Apache](#step-1-installing-apache)
- [Step 2: Managing the Apache Service](#step-2-managing-the-apache-service)
- [Step 3: Setting Up a Virtual Host](#step-3-setting-up-a-virtual-host)
- [Virtual Host Configuration Explained](#virtual-host-configuration-explained)
- [Verification](#verification)
- [Troubleshooting](#troubleshooting)
- [Key Takeaways](#key-takeaways)
- [Author](#author)

---

## Overview

Apache is one of the most widely used web servers in the world. It offers dynamically loadable modules, strong media support, and integrates cleanly with other software.

In this lab, you will:

1. Install Apache on Ubuntu and confirm it is serving the default page.
2. Control the Apache process using `systemctl`.
3. Host a custom website using a **Virtual Host**, Apache's mechanism for serving multiple sites from a single server.

## Skills Demonstrated

- Linux fundamentals and package management with `apt`
- Service management with `systemd` / `systemctl`
- Web server configuration (Apache2 `sites-available` / `sites-enabled` workflow)
- File and directory permissions
- Configuration validation and basic troubleshooting

## Prerequisites

| Requirement | Details |
|---|---|
| Operating system | Ubuntu Linux virtual machine |
| Privileges | Administrative access (`sudo`) |
| Network | Port **80** must be free |

Stop any service that may already be using port 80:

```bash
sudo systemctl stop nginx
sudo systemctl stop apache2
```

> **Note:** If either service is not installed, `systemctl` will report that the unit was not found. This is safe to ignore.

---

## Step 1: Installing Apache

### 1.1 Update package lists

```bash
sudo apt update -y
```

### 1.2 Install Apache

```bash
sudo apt install apache2 -y
```

This installs Apache and all required dependencies.

### 1.3 Verify the installation

Apache starts automatically after installation. Check its status:

```bash
sudo systemctl status apache2
```

A healthy service reports `active (running)`.

Then browse to the server:

```
http://<server_ip_address>
```

You should see the default **Apache2 Ubuntu Default Page**.

---

## Step 2: Managing the Apache Service

| Action | Command |
|---|---|
| Stop | `sudo systemctl stop apache2` |
| Start | `sudo systemctl start apache2` |
| Restart | `sudo systemctl restart apache2` |
| Reload (apply config changes without dropping connections) | `sudo systemctl reload apache2` |
| Disable on boot | `sudo systemctl disable apache2` |
| Enable on boot | `sudo systemctl enable apache2` |

---

## Step 3: Setting Up a Virtual Host

By default, Apache serves content from `/var/www/html`, which is fine for a single site. To host multiple websites on one server, use Virtual Hosts.

> Replace `your_domain` with your actual domain name throughout this section.

### 3.1 Create the website directory

```bash
sudo mkdir -p /var/www/your_domain
sudo chmod -R 755 /var/www/your_domain
```

### 3.2 Create an index page

```bash
sudo nano /var/www/your_domain/index.html
```

```html
<html>
    <head>
        <title>Welcome to Your Domain!</title>
    </head>
    <body>
        <h1>Success! The your_domain virtual host is working!</h1>
    </body>
</html>
```

### 3.3 Create the virtual host configuration

```bash
sudo nano /etc/apache2/sites-available/your_domain.conf
```

```apache
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    ServerName your_domain
    ServerAlias www.your_domain
    DocumentRoot /var/www/your_domain
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

### 3.4 Enable the site and validate

```bash
# Enable the new site
sudo a2ensite your_domain.conf

# Disable the default site
sudo a2dissite 000-default.conf

# Check for configuration errors
sudo apache2ctl configtest
```

Expected output:

```
Syntax OK
```

Restart Apache to apply the changes:

```bash
sudo systemctl restart apache2
```

---

## Virtual Host Configuration Explained

A Virtual Host tells Apache: *"when a request comes in for this name, serve files from this folder instead of the default site."*

| Directive | Purpose |
|---|---|
| `<VirtualHost *:80>` | Answers requests on port 80 (standard HTTP) from any IP address on the server. |
| `ServerName` | The primary domain this site responds to. |
| `ServerAlias` | Additional names for the same site, such as the `www` variant. |
| `DocumentRoot` | The directory containing the site's files. Apache typically serves `index.html` first. |
| `ErrorLog` / `CustomLog` | Per-site error and access logs, keeping troubleshooting and traffic tracking separate from other sites. |

> **Gotcha:** `ServerName` and `ServerAlias` must match the domain actually used in the browser or DNS. Otherwise Apache falls back to the default site instead of yours.

---

## Verification

Display the parsed virtual host configuration and confirm your site is listed:

```bash
sudo apache2ctl -S
```

Then confirm the service is healthy and the site responds:

```bash
sudo systemctl status apache2
curl -I http://your_domain
```

If the domain does not resolve yet (for example, in a local VM), add a temporary entry to `/etc/hosts` on your client machine:

```
<server_ip_address>  your_domain www.your_domain
```

---

## Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| Apache fails to start | Port 80 is in use by another service | Run `sudo ss -tlnp \| grep :80` and stop the conflicting service |
| Default page shows instead of your site | `ServerName`/`ServerAlias` mismatch, or the site is not enabled | Check `apache2ctl -S`, confirm `a2ensite` was run, and verify the domain |
| `Syntax error` on `configtest` | Typo in the `.conf` file | Re-check directives and closing tags in `your_domain.conf` |
| `403 Forbidden` | Incorrect permissions on the web root | Re-apply `chmod -R 755 /var/www/your_domain` |
| Changes not appearing | Config not reloaded | Run `sudo systemctl reload apache2` |
| Logs | Errors and access records | `sudo tail -f /var/log/apache2/error.log` |

---

## Key Takeaways

- Apache's `sites-available` / `sites-enabled` layout, managed with `a2ensite` and `a2dissite`, keeps configuration modular and reversible.
- Always run `apache2ctl configtest` before restarting to catch syntax errors safely.
- Use `reload` instead of `restart` to apply configuration changes without interrupting active connections.
- `apache2ctl -S` is the quickest way to see how Apache is resolving your virtual hosts.

---

## Author

**Gerald Oti** - DevOps / Platform / SRE Engineer, Berlin

- GitHub: [goti13](https://github.com/goti13)
- LinkedIn: [gerald-oti](https://www.linkedin.com/in/gerald-oti/)
