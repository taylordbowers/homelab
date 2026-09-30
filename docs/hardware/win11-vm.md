# Windows 11 VM

A Windows 11 Pro VM for the occasional Windows-only browser task or app that doesn't run on Linux.

## Setup

- **VM ID:** 301 on `pve`
- **IP:** 10.0.0.18 (fixed)
- **Type:** Q35 / UEFI (OVMF)
- **OS:** Windows 11 Pro
- **CPU:** 2 cores
- **RAM:** 4 GB
- **Disk:** 64 GB on `flash` ZFS
- **Guest agent:** QEMU guest agent installed
- **Autostart:** yes

## Access

Reached through [Apache Guacamole](../services/management/guacamole.md) (CT 105) over RDP. RDP (TCP 3389) is reachable only inside the LAN. NLA is disabled so Guacamole can authenticate with a username and password directly.

## Notes

- Windows Firewall blocks ICMP by default, so the VM doesn't answer pings.
- It's deliberately left out of Uptime Kuma's monitoring-coverage check, since nothing depends on it.
- It's included in the nightly backup (about 35GB compressed).
