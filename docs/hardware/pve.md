# pve — The Node

**The only node.** It's a mobile workstation running headless with the lid closed, side-mounted on the 10-inch rack. Since October 2026 it hosts every workload in the lab, plus the RAIDZ1 array.

## Specs

| Component | Detail |
|---|---|
| **CPU** | Intel Core i9-11950H (8 cores / 16 threads) |
| **RAM** | 64GB |
| **GPU** | NVIDIA RTX A3000 Laptop GPU (6GB VRAM, Ampere), shared by LXC device passthrough to CT 101, CT 103, and CT 104 |
| **Disk** | 1TB NVMe: 149GB LVM for Proxmox, the rest is the `flash` ZFS pool (800GB). A low-profile heatsink replaces the bottom cover's cooling |
| **SATA** | ASM1166 6-port M.2→SATA card in a spare M.2 slot (native AHCI, per-drive SMART) |
| **Array** | 4x 8TB WD Red Plus RAIDZ1 (`tank_new`) in the rack bays, on their own ATX power supply |
| **Proxmox** | VE 9.2 (Debian 13), kernel 6.17 pinned for the NVIDIA driver |

## ZFS Pools

### `flash`: NVMe pool
A single-partition ZFS pool that holds every LXC rootfs and VM disk.

!!! warning
    It's single-disk ZFS with no redundancy. That's why every guest is backed up nightly to the RAIDZ1 array. See [Backups](../infrastructure/backups.md).

### `tank_new`: RAIDZ1 bulk pool
Four 8TB WD Red Plus drives in RAIDZ1, which survives a single drive failure. It's about a third full. A monthly scrub runs through the standard ZFS timer. The pool moved here from pve-guide in October 2026 (exported there, imported here). See [Storage](../infrastructure/storage.md).

| Dataset | Used for |
|---|---|
| `subvol-200-disk-0` | Media, photos, Nextcloud data, game-server data. Mounted in CT 200 and CT 121, bind-mounted into CT 101 and CT 104 |
| `backup` | Nightly vzdump archives (see [Backups](../infrastructure/backups.md)) |

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
| CT 102 | claude | LXC | Claude Code agent + automation services |
| CT 103 | jellyfin | LXC | Jellyfin, Jellyseerr, Jellystat (GPU enabled) |
| CT 104 | immich | LXC | Immich + GPU speech-to-text for voice agents (GPU enabled) |
| CT 105 | guacamole | LXC | Apache Guacamole (browser RDP/VNC) |
| CT 106 | vaultwarden | LXC | Password manager (private) |
| CT 107 | jarvis | LXC | Jarvis HUD + voice assistant |
| CT 121 | portainer | LXC | Docker host for NPM, Uptime Kuma, Homarr, Crafty, Restreamer, Cloudflare DDNS, Portainer |
| CT 200 | media | LXC | Samba file server for the 22TB `/data` dataset |
| VM 119 | mediaServer | VM | *arr stack + VPN download clients |
| VM 300 | amp-gameserver | VM | AMP multi-game server |
| VM 301 | windows11-browser | VM | Windows 11 Pro for Windows-only tasks |

## Host services

- **Tailscale**: subnet router + exit node for remote access (see [Tailscale](../services/network/tailscale.md))
- **Filebrowser**: host-level file manager on port 8085
- **Postfix → Gmail relay**: sends backup-failure and system alerts
- **No NFS.** It was removed in October 2026, once every consumer had a local mount
