# Hardware Overview

Since October 2026 the lab runs on **one** physical machine: a laptop running Proxmox VE 9.2, with the RAIDZ1 array in a 10-inch rack beside it.

## The node

| | pve |
|---|---|
| **IP** | 10.0.0.3 |
| **Form factor** | Mobile workstation (laptop), side-mounted on the rack with the bottom panel off |
| **CPU** | Intel Core i9-11950H (8c/16t) |
| **RAM** | 64GB |
| **GPU** | NVIDIA RTX A3000 Laptop (6GB), shared to 3 LXCs |
| **Boot / OS** | 1TB NVMe (149GB LVM for Proxmox) |
| **Guest storage** | `flash` ZFS, 800GB on the same NVMe |
| **Bulk storage** | 4x 8TB WD Red Plus RAIDZ1 (`tank_new`, ~21TB usable) in two printed 1U hot-swap bays, on an ASM1166 M.2→SATA card |
| **Guests** | 11 LXCs + 3 VMs (everything) |

See [pve](pve.md) for details.

## Storage Architecture

```
pve
├── nvme0n1 (1TB NVMe)
│   ├── LVM (149GB)  → Proxmox root + swap
│   └── flash ZFS (800GB) → all LXC rootfs + VM disks
└── ASM1166 M.2→SATA card (6 ports, 4 used)
    └── sda–sdd (4x 8TB WD Red Plus) → tank_new RAIDZ1
        ├── subvol-200-disk-0  → media + photos + game data (/data)
        └── backup             → nightly vzdump target
```

## The rack

A printed 10-inch, 12U rack, bottom to top:

| U | What |
|---|---|
| 1–2 | Two 1U bays, 2x 3.5" each, for the four array drives (Dell caddies, SATA couplers as the backplane) |
| 3–4 | A reused ATX power supply for the drives in a printed 2U mount. Its fan also pulls air over the bays |
| 5 | PDU |
| 6–10 | Free |
| 11 | Switch |
| 12 | Patch panel |

The laptop hangs on the side with its bottom panel facing in, so the SATA cables run straight from the M.2 card into the bays. A low-profile heatsink replaces the cooling the bottom cover used to give the boot NVMe. The drives run at about 30 °C.

## Network

The node sits on a flat `/24` home network behind the ISP gateway (`10.0.0.254`). There are no VLANs. AdGuard Home handles internal DNS, and Tailscale on `pve` provides remote access. See [Network](../infrastructure/network.md).

## History

| Period | Layout |
|---|---|
| to Jul 2026 | **pve-guide** (Xeon tower: storage + infra) + **pve2** (i7-4790 / 32GB / GTX 980 Ti, the GPU node for Jellyfin, Immich, and Nextcloud) |
| Jul 2026 | **pve** (laptop) joins. Every guest on pve2, plus most of pve-guide's, is migrated to it and verified. pve2 is decommissioned and the GTX 980 Ti retired |
| Sep 2026 | The lab moves house. `pve` came up first and pve-guide followed; all guest IPs were re-pinned on the new network |
| Oct 2026 | The four array drives move from pve-guide into the rack on `pve`. The last two containers follow, NFS is replaced with local mounts, and pve-guide is powered off. See [pve-guide](pve-guide.md) |
| Oct 2026 | pve-guide is removed from the cluster, and `pve` is upgraded in place to Proxmox VE 9.2 (Debian 13) |
| Planned | Rebuild pve-guide as an off-site Proxmox Backup Server at a family member's house |

The laptop draws a fraction of the old towers' power, and its battery works as a built-in UPS. See [Power & Resilience](../infrastructure/power.md).
