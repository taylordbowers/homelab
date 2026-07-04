# Hardware Overview

Two physical servers running Proxmox VE in a cluster, connected over a standard home network.

## Nodes at a Glance

| | pve-guide (Node 1) | pve2 (Node 2) |
|---|---|---|
| **IP** | 10.0.0.1 | 10.0.0.2 |
| **CPU** | Intel Xeon E3-1245 v3 | Intel Core i7-4790 |
| **Cores** | 8 threads @ 3.4GHz | 8 threads @ 3.6GHz |
| **RAM** | 32GB | 32GB |
| **GPU** | Intel iGPU (→ mediaServer) | NVIDIA GTX 980 Ti 6GB |
| **OS Drive** | 465GB SSD | 932GB SSD |
| **Fast Storage** | 932GB SSD (`flash` ZFS) | 932GB SSD (`flash` ZFS) |
| **Bulk Storage** | 4x 7.3TB WD Red RAIDZ1 (`tank_new`) | 3.6TB HDD |
| **Proxmox** | Primary node | Secondary node |

## Storage Architecture

```
pve-guide
├── /dev/sda  (465GB SSD)  → Proxmox OS (LVM)
├── /dev/sdf  (932GB SSD)  → flash ZFS pool (container rootfs, docker volumes)
└── /dev/sdb-sde (4x 7.3TB WD Red) → tank_new RAIDZ1 (~17TB usable)
    └── subvol-200-disk-0 (22TB quota) → media LXC data mount

pve2
├── /dev/sda  (932GB SSD)  → Proxmox OS (LVM) + flash ZFS pool
└── /dev/sdb  (3.6TB HDD)  → additional storage
```

## Network

Both nodes connect to the same `/24` home network (`10.0.0.0/24`) via the main router at `10.0.0.254`. No VLANs — flat network with AdGuard Home handling DNS.
