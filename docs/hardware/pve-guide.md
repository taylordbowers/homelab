# pve-guide — Storage Node

**Storage node.** This is the original homelab server. It now mainly holds the 4-drive RAIDZ1 array, serves it over NFS and SMB, receives the nightly backups, and runs the two containers that mount the array directly.

!!! note "Retiring"
    Everything that could move has been migrated to [pve](pve.md). CT 121 and CT 200 stay here because they mount `tank_new` directly. They'll move once the array has a new home, such as a DAS or HBA enclosure on `pve`, and then this box can retire.

## Specs

| Component | Detail |
|---|---|
| **CPU** | Intel Xeon E3-1245 v3 @ 3.40GHz (4c/8t) |
| **RAM** | 32GB |
| **GPU** | Intel HD Graphics P4600 (iGPU, unused) |
| **OS Disk** | 500GB HDD → Proxmox root (LVM) |
| **Flash Pool** | 1TB HDD → `flash` ZFS (single disk; the name is historical) |
| **Bulk Pool** | 4x 8TB WD Red Plus → `tank_new` RAIDZ1 (~21TB usable) |
| **Proxmox** | VE 8.4 (Debian 12) |

## ZFS Pools

### `tank_new`: RAIDZ1 bulk pool
Four 8TB WD Red Plus drives in RAIDZ1, which survives a single drive failure. It's about a third full. A monthly scrub runs through the standard ZFS timer, and the last one found 0 errors.

| Dataset | Used for |
|---|---|
| `subvol-200-disk-0` | Media, photos, Nextcloud data, game-server data. Mounted at `/data` in CT 200 and CT 121, NFS-exported to `pve` |
| `backup` | Nightly vzdump archives for **both** nodes (see [Backups](../infrastructure/backups.md)) |

### `flash`: single-disk pool
Rootfs for CT 121 and CT 200. Despite the name, it's a spinning disk.

## Exports

| Export | Protocol | Consumer |
|---|---|---|
| `tank_new/subvol-200-disk-0` | NFS | `pve` host (→ Nextcloud CT 101 bind mount) |
| `tank_new/subvol-200-disk-0/immich` | NFS | Immich (CT 104) |
| `tank_new/backup` | NFS | `pve` vzdump target |
| `/data` (from CT 200) | SMB | Jellyfin (CT 103), mediaServer (VM 119) |

## Containers

| ID | Name | Type | Role |
|---|---|---|---|
| CT 121 | portainer | LXC | Docker host for NPM, Uptime Kuma, Homarr, Crafty, Restreamer, Cloudflare DDNS, Portainer |
| CT 200 | media | LXC | Samba file server for the 22TB `/data` dataset |

## Host services

- **Filebrowser**: host-level file manager on port 8085
- **NFS server**: exports listed above
- **Postfix → Gmail relay**: backup-failure and system alerts
