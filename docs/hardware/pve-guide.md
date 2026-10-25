# pve-guide — Retired Storage Node

**Retired.** This is the original homelab server. Until October 2026 it held the 4-drive RAIDZ1 array, served it to the laptop over NFS and SMB, received the nightly backups, and ran the two containers that mounted the array directly.

!!! success "Powered off — 2026-10-24"
    The four array drives moved into the 10-inch rack on [pve](pve.md). CT 121 (portainer) and CT 200 (media) moved with them. Every NFS consumer was switched to a local mount, and this box was powered off. It stays a cluster member for a few days as a rollback path, and then it's removed from the cluster.

## How the move was done

1. **Prep without downtime.** An M.2→SATA card went into the laptop. A safety copy of the photo library went onto the laptop's NVMe, so the array wasn't the only copy during the move. The two containers' root disks were pre-copied with `zfs send` while they ran. A fresh scrub and SMART check ran the night before. An automated check emailed "ready" once every step had passed.
2. **The window (~1 hour).** Stop the guests that use the array. Send a final incremental copy of the container disks. Then `zpool export` here, move the drives, `zpool import` on `pve`, and repoint the Proxmox storage config.
3. **Rewire.** Nextcloud and Immich went from NFS to bind mounts. The game-server VM's data moved onto its own disk. The backup job went from NFS to the local dataset. The NFS server was removed.

!!! tip "Lessons"
    - `zfs send` needs `-p` for container disks. Without it, the dataset properties Proxmox relies on (`acltype=posix`, `xattr=sa`) don't come across.
    - With two nodes and one powered off, the survivor has no quorum. Proxmox won't start any guests after a reboot until `pvecm expected 1` is run, and that resets every boot. Giving the survivor 2 of 3 votes for the rollback window makes reboots unattended-safe.
    - Bay and port order don't matter to ZFS. Label the caddies with drive serials anyway, so a failed disk can be found later.

## Specs (for the record)

| Component | Detail |
|---|---|
| **CPU** | Intel Xeon E3-1245 v3 @ 3.40GHz (4c/8t) |
| **RAM** | 32GB |
| **GPU** | Intel HD Graphics P4600 (iGPU, unused) |
| **OS Disk** | 500GB HDD → Proxmox root (LVM) |
| **Flash Pool** | 1TB HDD → `flash` ZFS (single disk; the name is historical) |
| **Proxmox** | VE 8.4 (Debian 12) when retired; removed from the cluster 2026-10-25 |

## What's next

The plan is to wipe it and rebuild it as an **off-site Proxmox Backup Server** at a family member's house, with a new mirrored pair of drives. It would hold the first copy of the lab outside the house. See [Backups → Roadmap](../infrastructure/backups.md#roadmap).
