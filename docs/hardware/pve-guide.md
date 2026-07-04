# pve-guide — Node 1

**Primary node.** Hosts the bulk storage and most of the infrastructure/network services.

## Specs

| Component | Detail |
|---|---|
| **CPU** | Intel Xeon E3-1245 v3 @ 3.40GHz (4c/8t) |
| **RAM** | 32GB |
| **GPU** | Intel HD Graphics P4600 (iGPU, passed through to mediaServer VM) |
| **OS Disk** | 465GB SSD → Proxmox root (LVM) |
| **Flash Pool** | 932GB SSD → `flash` ZFS (single disk) |
| **Bulk Pool** | 4x 7.3TB WD Red Pro → `tank_new` RAIDZ1 (~17TB usable) |

## ZFS Pools

### `flash` — SSD pool
Used for container rootfs volumes and Docker data directories. Fast random I/O for running services.

!!! warning
    Single-disk ZFS — no redundancy. Backups are important for anything on this pool.

### `tank_new` — RAIDZ1 bulk pool
Four 7.3TB WD Red drives in RAIDZ1. Hosts all media, photos, and cloud data. Mounted into the `media` LXC. Periodic scrubs run via the standard ZFS systemd timer.

## Containers & VMs

| ID | Name | Type | Role |
|---|---|---|---|
| CT 100 | adguard | LXC | DNS + ad blocking |
| CT 106 | vaultwarden | LXC | Password manager (Bitwarden-compatible) |
| CT 121 | portainer | LXC | Docker management + reverse proxy + dashboard + monitoring |
| CT 200 | media | LXC | 22TB data store (NFS/SMB-exported to other VMs) |
| VM 119 | mediaServer | VM | *arr stack + VPN download clients |
