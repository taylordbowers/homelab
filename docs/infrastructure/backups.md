# Backups

Every guest on both nodes gets a nightly Proxmox `vzdump` backup to the RAIDZ1 array.

## Jobs

Two scheduled jobs, one per node, because each node writes to its own storage ID:

| Node | Storage | Target |
|---|---|---|
| pve | `backup-nfs` (NFS) | `tank_new/backup` on pve-guide, over the network |
| pve-guide | `backup` (dir) | `tank_new/backup`, local |

| Setting | Value |
|---|---|
| Schedule | Daily 02:00 |
| Scope | All guests |
| Mode | Snapshot (no downtime) |
| Compression | zstd |
| Retention | 7 daily · 4 weekly · 3 monthly |
| Alerts | Email on failure (Postfix → Gmail relay on each node) |

The full set is 13 guests and about 140GB per night after compression. `pve`'s guests end up on different physical disks in a different machine from where they run.

## What's excluded

The 22TB media dataset (`tank_new:subvol-200-disk-0`, mounted in CT 200 and CT 121) is marked `backup=0`. Backing the array up onto the same array protects against nothing, and three rotated copies wouldn't fit. Its protection for now is RAIDZ1 plus scrubs, which is **not** a backup.

## Verification

- A weekly health-audit email reports the newest backup for each guest and flags any guest with no backup, or one older than 8 days.
- Failure mail was tested end-to-end through Proxmox's notification test.

## Lessons learned

!!! warning "Host-owned files inside an unprivileged CT fail the whole backup"
    If files owned by host uid 0 end up inside an **unprivileged** container's subvolume (for example, staged from the host), `tar` exits with an error and vzdump discards the whole archive for that guest. Find them with `find <subvol> -uid -100000` and `chown` only those paths to the container's mapped root (`100000:100000`). Never do it recursively.

!!! warning "`pct set` on a running container is staged, not applied"
    Changing mount-point flags such as `backup=0` with `pct set` on a **running** CT stages the change as *pending*. `pct config` shows the pending value, so it looks applied, but vzdump uses the active config. For metadata-only flags, edit `/etc/pve/lxc/<id>.conf` directly, or restart the CT.

- Docker-heavy containers such as CT 121 take about 20 minutes, because the overlay2 layers are millions of small files.

## Roadmap

- **Off-site copy.** The plan is a Proxmox Backup Server at a family member's house, seeded on the LAN and then synced over Tailscale. It would be the first real protection for the media dataset.
