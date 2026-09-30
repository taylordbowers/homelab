# pve — Primary Node

**Primary node.** It's a mobile workstation running headless with the lid closed, and it hosts nearly every workload in the lab, including everything that needs the GPU.

## Specs

| Component | Detail |
|---|---|
| **CPU** | Intel Core i9-11950H (8 cores / 16 threads) |
| **RAM** | 64GB |
| **GPU** | NVIDIA RTX A3000 Laptop GPU (6GB VRAM, Ampere), shared by LXC device passthrough to CT 101, CT 103, and CT 104 |
| **Disk** | 1TB NVMe: 149GB LVM for Proxmox, the rest is the `flash` ZFS pool (800GB) |
| **Proxmox** | VE 8.4 (Debian 12) |

## ZFS Pools

### `flash`: NVMe pool
A single-partition ZFS pool that holds every LXC rootfs and VM disk on this node.

!!! warning
    It's single-disk ZFS with no redundancy. That's why every guest is backed up nightly to the RAIDZ1 array on pve-guide over NFS. See [Backups](../infrastructure/backups.md).

## Laptop-as-server tweaks

- **Lid and sleep disabled** so the machine runs closed and headless
- **Battery charge capped at 80%** for longevity, since it's always on AC
- **AC power watchdog** treats the battery as a UPS and shuts the node down gracefully during a long outage (see [Power & Resilience](../infrastructure/power.md))
- **NVIDIA device-node service** makes sure the GPU nodes exist before guests autostart (see [GPU Passthrough](../infrastructure/gpu-passthrough.md#boot-ordering))

## Containers & VMs

| ID | Name | Type | Role |
|---|---|---|---|
| CT 100 | adguard | LXC | DNS, ad blocking, internal rewrites |
| CT 101 | nextcloud | LXC | Nextcloud AIO (GPU enabled) |
| CT 102 | claude | LXC | Claude Code agent, automation services, legacy wiki |
| CT 103 | jellyfin | LXC | Jellyfin, Jellyseerr, Jellystat (GPU enabled) |
| CT 104 | immich | LXC | Immich + GPU speech-to-text for voice agents (GPU enabled) |
| CT 105 | guacamole | LXC | Apache Guacamole (browser RDP/VNC) |
| CT 106 | vaultwarden | LXC | Password manager (private) |
| CT 107 | jarvis | LXC | Jarvis HUD + voice assistant |
| VM 119 | mediaServer | VM | *arr stack + VPN download clients |
| VM 300 | amp-gameserver | VM | AMP multi-game server |
| VM 301 | windows11-browser | VM | Windows 11 Pro for Windows-only tasks |

## Host services

- **Tailscale**: subnet router + exit node for remote access (see [Tailscale](../services/network/tailscale.md))
- **Filebrowser**: host-level file manager on port 8085
- **Postfix → Gmail relay**: sends backup-failure and system alerts
- **NFS client**: mounts media, photos, and the backup target from pve-guide
