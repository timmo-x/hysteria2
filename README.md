Hysteria2 is a high-performance, UDP-based proxy and tunneling protocol designed to operate reliably on unstable, high-latency, or heavily restricted networks. Built on modern transport techniques and TLS encryption, it focuses on maintaining throughput and connection stability where traditional TCP-based solutions struggle.

Its traffic patterns closely resemble standard QUIC/HTTPS over UDP, allowing Hysteria2 connections to appear similar to normal web traffic under deep packet inspection (DPI). This makes protocol identification, throttling, and selective blocking significantly more difficult in environments that rely on traffic analysis or protocol-based filtering.


# Hysteria2 — VPS Installation & Setup Guide

This document provides a **step-by-step guide** to install and configure **Hysteria2** on a VPS using:
**Fedora / RHEL-like systems**

---

## Prerequisites

Before starting, ensure you have:

- A VPS with a public IPv4 address
- A domain or subdomain you control
- DNS access to create A records
- Root or sudo access on the server

### Placeholders Used
- `domain.com` → your actual domain (e.g. `node001.m0n.org`)
- `1.2.3.4` → your VPS public IP

---

## Step 1 — Set DNS to Your VPS

Create an **A record** pointing your domain to the VPS IP.

Example:

```
node001.m0n.org  -->  1.2.3.4
```

---

## Step 2 — Install Hysteria2

```bash
bash <(curl -fsSL https://get.hy2.sh/)
```

---

## Step 3 — Configure Firewall

```bash
firewall-cmd --permanent --add-port=80/tcp
firewall-cmd --permanent --add-port=443/tcp
firewall-cmd --permanent --add-port=443/udp
firewall-cmd --reload
```

---

## Step 4 — Create TLS Certificates

```bash
dnf install -y certbot

certbot certonly --standalone --agree-tos --email timmo@domain.com -d domain.com
```

```bash
mkdir -p /etc/hysteria

cp /etc/letsencrypt/live/domain.com/fullchain.pem /etc/hysteria/
cp /etc/letsencrypt/live/domain.com/privkey.pem /etc/hysteria/

chown hysteria:hysteria /etc/hysteria/fullchain.pem /etc/hysteria/privkey.pem
chmod 640 /etc/hysteria/fullchain.pem /etc/hysteria/privkey.pem
```

## Automatic Certificate Renewal (Certbot Hook)

Since the TLS certificates are copied manually into `/etc/hysteria`, a **Certbot deploy hook** is required to ensure they are updated automatically after renewal.

### Create the Renewal Hook Script

Create a deploy hook that runs after every successful certificate renewal:

```bash
vim /etc/letsencrypt/renewal-hooks/deploy/hysteria.sh
```

Add the following content (replace the domain name as needed):

```bash
#!/bin/bash

cp /etc/letsencrypt/live/domain.com/fullchain.pem /etc/hysteria/
cp /etc/letsencrypt/live/domain.com/privkey.pem /etc/hysteria/

chown hysteria:hysteria /etc/hysteria/fullchain.pem /etc/hysteria/privkey.pem
chmod 640 /etc/hysteria/fullchain.pem /etc/hysteria/privkey.pem
```

### Make the Script Executable

```bash
chmod +x /etc/letsencrypt/renewal-hooks/deploy/hysteria.sh
```

---

## Step 5 — Configure Hysteria2

```bash
mkdir -p /etc/hysteria
vim /etc/hysteria/config.yaml
```

Use the online config generator:  
https://m0n.org/toolz/hysteriagen.html

---

## Step 6 — Start and Enable Service

```bash
systemctl enable --now hysteria-server.service
systemctl status hysteria-server.service
```

---

## Step 7 — SELinux (If Needed)

```bash
semanage fcontext -a -t bin_t '/usr/local/bin/hysteria'
restorecon -v /usr/local/bin/hysteria
```

---

## Client Setup

Use the config generator to scan the QR code or download client configs:  
https://m0n.org/toolz/hysteriagen.html

---

## Troubleshooting

```bash
journalctl -u hysteria-server.service -e
```
