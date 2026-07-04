# Windows 11 VM

A Windows 11 Pro VM kept around for the occasional Windows-only browser task or app that doesn't run on Linux.

## Setup

- **VM ID:** 301 on pve2
- **Type:** Q35/UEFI VM
- **OS:** Windows 11 Pro
- **CPU:** 2 cores
- **RAM:** 4 GB
- **Storage:** flash ZFS subvolume

## Access

Reached via [Apache Guacamole](../services/management/guacamole.md) (CT 105) over RDP. RDP is exposed on TCP 3389 inside the LAN only. NLA disabled (so Guacamole can authenticate with username/password directly).

## Notes

- Windows Firewall blocks ICMP by default, so it doesn't respond to ping. Uptime Kuma monitors it via TCP port check on 3389 instead.
- Spinning down when not in use saves resources on pve2. The VM is started on demand.
