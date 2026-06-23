# Linux NTP Time Synchronization Setup

A practical guide to configuring and verifying NTP time synchronization on Linux servers using either **systemd-timesyncd** (default, no install) or **chrony** (recommended for production).

## Table of Contents

1. [Check Current Time Status](#1-check-current-time-status)
2. [Method 1: systemd-timesyncd](#method-1-systemd-timesyncd-no-installation-required)
3. [Method 2: chrony](#method-2-chrony-recommended-for-servers)
4. [Time Zone Configuration](#time-zone-configuration)
5. [Hardware Clock Sync](#hardware-clock-sync-optional)
6. [Troubleshooting](#troubleshooting)
7. [Verification Checklist](#verification-checklist)

---

## 1. Check Current Time Status

Run this before making any changes:

```bash
timedatectl status
date -u
```

Key fields to look at:

- `System clock synchronized` — should be `yes`
- `NTP service` — should be `active`
- `Time zone` — should match your server's actual location

---

## Method 1: systemd-timesyncd (No Installation Required)

This is the default time sync service on most Ubuntu/Debian systems.

### Step 1 — Enable NTP

```bash
sudo timedatectl set-ntp true
```

### Step 2 — Configure NTP server

```bash
sudo nano /etc/systemd/timesyncd.conf
```

```ini
[Time]
NTP=ntp.day.ir
FallbackNTP=ntp.ubuntu.com
```

You can specify multiple servers (space-separated):

```ini
NTP=ntp.day.ir ntp.ubuntu.com pool.ntp.org
```

### Step 3 — Restart the service

```bash
sudo systemctl restart systemd-timesyncd
```

### Step 4 — Verify synchronization

```bash
timedatectl status
timedatectl show-timesync --all
```

Expected output:

```text
System clock synchronized: yes
NTP service: active
ServerName=ntp.day.ir
```

### Step 5 — Force a re-sync (if needed)

```bash
sudo timedatectl set-ntp false
sudo timedatectl set-ntp true
sudo systemctl restart systemd-timesyncd
```

---

## Method 2: chrony (Recommended for Servers)

Chrony handles network jitter and intermittent connectivity better than timesyncd, making it the better choice for production servers and monitoring-sensitive environments.

### Step 1 — Install chrony

```bash
sudo apt update
sudo apt install chrony -y
```

> `systemd-timesyncd` and `chrony` should not run at the same time. Installing chrony on Ubuntu/Debian typically disables timesyncd automatically — confirm with `systemctl status systemd-timesyncd`.

### Step 2 — Configure NTP servers

```bash
sudo nano /etc/chrony/chrony.conf
```

Add or replace the server lines:

```text
server ntp.day.ir iburst
server ntp.ubuntu.com iburst
server pool.ntp.org iburst
```

### Step 3 — Restart and enable the service

```bash
sudo systemctl restart chrony
sudo systemctl enable chrony
```

### Step 4 — Check sync status

```bash
chronyc tracking
chronyc sources -v
```

`chronyc tracking` shows current offset and stratum; `chronyc sources -v` shows which servers are reachable and which one is selected (marked with `*`).

### Step 5 — Verify system time

```bash
timedatectl status
date -u
```

---

## Time Zone Configuration

List available time zones if you're not sure of the exact name:

```bash
timedatectl list-timezones | grep -i tehran
```

Set the correct time zone:

```bash
sudo timedatectl set-timezone Asia/Tehran
```

Or, for Azerbaijan:

```bash
sudo timedatectl set-timezone Asia/Baku
```

---

## Hardware Clock Sync (Optional)

Sync the system (software) clock to the hardware clock — useful before a reboot or on systems without a battery-backed RTC sync:

```bash
sudo hwclock --systohc
```

---

## Troubleshooting

### System clock not synchronized

```bash
timedatectl status
```

Restart the active service:

```bash
sudo systemctl restart systemd-timesyncd
# or
sudo systemctl restart chrony
```

### NTP server not responding

Check basic network connectivity:

```bash
ping -c 4 8.8.8.8
```

Check DNS resolution:

```bash
cat /etc/resolv.conf
```

### Firewall blocking NTP

NTP uses **UDP port 123**. Make sure it's open both outbound and (if this server acts as an NTP source) inbound:

```bash
sudo ufw allow 123/udp
```

For `firewalld`-based systems:

```bash
sudo firewall-cmd --add-service=ntp --permanent
sudo firewall-cmd --reload
```

---

## Verification Checklist

- [ ] `System clock synchronized: yes`
- [ ] Correct time zone set
- [ ] NTP server reachable (`chronyc sources -v` or `timedatectl show-timesync --all`)

---

