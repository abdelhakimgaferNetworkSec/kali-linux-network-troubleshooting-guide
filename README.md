# Kali Linux NetworkManager eth0 Unmanaged Fix

A complete troubleshooting guide for fixing the common Kali Linux networking issue where `eth0` appears as **unmanaged**, receives no IP address, and has no Internet access inside VMware Workstation.

Keywords:

- Kali Linux
- NetworkManager
- eth0 unmanaged
- VMware Workstation
- DHCP
- No Internet Connection
- Linux Networking
- Kali Linux Network Troubleshooting

---

## Problem

After installing Kali Linux on VMware Workstation, the network interface appears available but does not receive an IP address.

Example:

```bash
nmcli device status
```

Output:

```text
DEVICE TYPE STATE CONNECTION
eth0 ethernet unmanaged --
```

Checking routes:

```bash
ip route
```

Output:

```text
172.17.0.0/16 dev docker0
172.18.0.0/16 dev br-xxxx
```

No default gateway exists.

---

## Symptoms

### Interface Exists

```bash
ip a
```

Output:

```text
eth0: state UP
```

### No IP Address

```bash
ip addr show eth0
```

Output:

```text
No inet address assigned
```

### NetworkManager Shows Unmanaged

```bash
nmcli device status
```

Output:

```text
eth0 ethernet unmanaged
```

---

## Root Cause

The NetworkManager configuration disables management of interfaces through ifupdown.

File:

```bash
/etc/NetworkManager/NetworkManager.conf
```

Problematic configuration:

```ini
[main]
plugins=ifupdown,keyfile

[ifupdown]
managed=false
```

Because of:

```ini
managed=false
```

NetworkManager ignores network interfaces.

---

## Solution

Open the configuration file:

```bash
sudo nano /etc/NetworkManager/NetworkManager.conf
```

Change:

```ini
managed=false
```

To:

```ini
managed=true
```

Final file:

```ini
[main]
plugins=ifupdown,keyfile

[ifupdown]
managed=true
```

---

## Restart NetworkManager

```bash
sudo systemctl restart NetworkManager
```

Verify:

```bash
systemctl status NetworkManager
```

---

## Check Device Status

```bash
nmcli device status
```

Expected:

```text
eth0 ethernet disconnected
```

or

```text
eth0 ethernet connected
```

Instead of:

```text
eth0 ethernet unmanaged
```

---

## Connect Interface

```bash
nmcli device connect eth0
```

---

## Verify IP Address

```bash
ip addr show eth0
```

Expected:

```text
inet 192.168.x.x/24
```

---

## Verify Default Route

```bash
ip route
```

Expected:

```text
default via 192.168.x.1 dev eth0
```

---

## VMware Checks

Verify:

### Network Adapter

- Connected
- Connect at power on

### Network Mode

- NAT
- Bridged

### VMware Services

Windows Services:

```text
VMware DHCP Service
VMware NAT Service
VMware Authorization Service
```

All services should be running.

---

## Useful Commands

### Show Interfaces

```bash
ip a
```

### Show Routes

```bash
ip route
```

### Show NetworkManager Devices

```bash
nmcli device status
```

### Restart NetworkManager

```bash
sudo systemctl restart NetworkManager
```

### Show NetworkManager Status

```bash
systemctl status NetworkManager
```

---

## Tested Environment

| Component | Version |
|------------|----------|
| Kali Linux | 2026.x |
| VMware Workstation | Tested |
| NetworkManager | Latest |
| Docker Installed | Yes |

---

## Tags

Kali Linux, VMware, NetworkManager, DHCP, Linux Networking, eth0 unmanaged, No Internet, VMware NAT, VMware Bridged, Network Troubleshooting

---

## License

MIT License
