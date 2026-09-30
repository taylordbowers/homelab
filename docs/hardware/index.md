# Hardware Overview

Two physical machines run Proxmox VE 8.4 as one cluster on a flat home network.

## Nodes at a Glance

| | pve (primary) | pve-guide (storage) |
|---|---|---|
| **IP** | 10.0.0.3 | 10.0.0.1 |
| **Form factor** | Mobile workstation (laptop) | Tower server |
| **CPU** | Intel Core i9-11950H (8c/16t) | Intel Xeon E3-1245 v3 (4c/8t) |
| **RAM** | 64GB | 32GB |
| **GPU** | NVIDIA RTX A3000 Laptop (6GB), shared to 3 LXCs | Intel iGPU (unused) |
| **Boot / OS** | 1TB NVMe (149GB LVM for Proxmox) | 500GB HDD (LVM) |
| **Guest storage** | `flash` ZFS, 800GB on the same NVMe | `flash` ZFS, 1TB HDD |
| **Bulk storage** | None (uses NFS from pve-guide) | 4x 8TB WD Red Plus RAIDZ1 (`tank_new`, ~21TB usable) |
| **Role** | Runs nearly every workload | Media array, backups, array-bound containers |
| **Guests** | 8 LXCs + 3 VMs | 2 LXCs |

## Storage Architecture

```
pve
└── nvme0n1 (1TB NVMe)
    ├── LVM (149GB)  → Proxmox root + swap
    └── flash ZFS (800GB) → all LXC rootfs + VM disks on this node

pve-guide
├── sda (500GB HDD)        → Proxmox OS (LVM)
├── sdf (1TB HDD)          → flash ZFS (CT 121 / CT 200 rootfs)
└── sdb–sde (4x 8TB WD Red Plus) → tank_new RAIDZ1
    ├── subvol-200-disk-0  → media + photos + game data (/data)
    └── backup             → nightly vzdump target for both nodes
```

## Network

Both nodes are on the same flat `/24` home network behind the ISP gateway (`10.0.0.254`). There are no VLANs. AdGuard Home handles internal DNS, and Tailscale on `pve` provides remote access. See [Network](../infrastructure/network.md).

## History

| Period | Layout |
|---|---|
| to Jul 2026 | **pve-guide** (storage + infra) + **pve2** (i7-4790 / 32GB / GTX 980 Ti, the GPU node for Jellyfin, Immich, and Nextcloud) |
| Jul 2026 | **pve** (laptop) joins. Every guest on pve2, plus most of pve-guide's, is migrated to it and verified. pve2 is decommissioned and the GTX 980 Ti retired |
| Sep 2026 | The lab moves house. `pve` came up first and pve-guide followed; all guest IPs were re-pinned on the new network |
| Planned | Move the RAIDZ1 array off pve-guide (DAS or HBA enclosure) so the old server can retire, possibly as an off-site backup box |

The laptop draws a fraction of the old towers' power, and its battery works as a built-in UPS. See [Power & Resilience](../infrastructure/power.md).
