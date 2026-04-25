# Ubuntu VPS Security Hardening

## Overview

This project documents the security hardening of a public-facing Ubuntu VPS used to host my personal portfolio website. The goal was to reduce unnecessary public exposure, improve firewall control, harden SSH access, and validate that required services remained available after the changes.

The project was performed on a live VPS environment, making it more realistic than a fully isolated lab. All changes were applied carefully to avoid breaking access to the website, Hestia control panel, SSH, and FTP workflow.

## Project Objectives

- Assess the initial security posture of a public Ubuntu VPS
- Identify exposed services and unnecessary open ports
- Enable and validate firewall controls
- Tighten Hestia firewall rules for unused mail services
- Harden SSH configuration
- Validate changes using external connectivity tests
- Document the process as a practical server hardening case study

## Environment

| Component | Details |
|---|---|
| Server Type | VPS |
| Operating System | Ubuntu 24.04 LTS |
| Hosting Provider | Contabo |
| Control Panel | Hestia Control Panel |
| Web Stack | Nginx, Apache, PHP-FPM |
| Database | MariaDB |
| Security Tools | UFW, Fail2Ban, Hestia Firewall |
| Main Use Case | Hosting a personal portfolio website |

## Initial Findings

During the baseline assessment, the server had multiple public-facing services enabled. The host firewall was inactive, and several ports related to mail, FTP, DNS, web services, SSH, and the control panel were listening.

Key observations:

- UFW firewall was inactive
- Multiple TCP services were publicly exposed
- Mail-related services were reachable from the internet
- Hestia firewall allowed IMAP, POP3, SMTP, FTP, DNS, web, SSH, and panel access
- SSH root login was enabled
- X11 forwarding was enabled
- Real SSH login attempts from external IP addresses appeared in authentication logs

## Screenshots

### 1. UFW inactive before hardening

The initial check showed that UFW was inactive.

![UFW inactive](screenshots/01-ufw-inactive.png)

### 2. Open ports before hardening

The baseline port review showed several public TCP services listening on the server.

![Open ports before hardening](screenshots/02-open-ports-before.png)

### 3. Hestia firewall before changes

Before hardening, Hestia allowed mail-related services such as IMAP, POP3, and SMTP.

![Hestia firewall before changes](screenshots/03-hestia-firewall-before.png)

## Hardening Actions

### 1. Enabled UFW firewall

UFW was enabled and configured with a default-deny inbound policy. Required services were allowed to avoid breaking access.

Allowed services:

| Service | Port |
|---|---|
| SSH | 22/tcp |
| HTTP | 80/tcp |
| HTTPS | 443/tcp |
| Hestia Panel | 8083/tcp |
| WireGuard | 51820/udp |

![UFW enabled](screenshots/04-ufw-enabled.png)

### 2. Updated Hestia firewall rules

The Hestia firewall was updated to block unused mail-related services while keeping required services available.

Changed to DROP:

| Service | Port |
|---|---|
| POP3 | 110, 995 |
| IMAP | 143, 993 |
| SMTP | 25, 465, 587 |

Kept as ACCEPT:

| Service | Port |
|---|---|
| SSH | 22 |
| Web | 80, 443 |
| Hestia Panel | 8083 |
| FTP | 21, 12000-12100 |
| DNS | 53 |

FTP was kept open because it was still needed for website editing.

![Hestia firewall after changes](screenshots/05-hestia-firewall-after.png)

### 3. Hardened SSH configuration

SSH was hardened by disabling direct root login, disabling X11 forwarding, and reducing the maximum authentication attempts.

Final SSH settings:

```text
PermitRootLogin no
MaxAuthTries 3
PubkeyAuthentication yes
PasswordAuthentication yes
X11Forwarding no
