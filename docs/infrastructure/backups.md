# Backups

Every guest gets a nightly Proxmox `vzdump` backup to the RAIDZ1 array.

## Job

One scheduled job covers every guest and writes to the `backup` storage, a directory on the local `tank_new/backup` dataset. Until October 2026 there were two jobs, one per node, and `pve`'s backups crossed the network over NFS. That ended when the array moved into `pve`.

| Setting | Value |
|---|---|
| Schedule | Daily 02:00 |
| Scope | All guests |
| Mode | Snapshot (no downtime) |
| Compression | zstd |
| Retention | 7 daily · 4 weekly · 3 monthly |
| Alerts | Email on failure (Postfix → Gmail relay) |

The full set is 14 guests and about 140GB per night after compression. Backups land on different physical disks (the HDD array) from where the guests run (the NVMe pool), but they're **in the same machine** now. A fire, theft, or power-supply fault would take both. That's why the off-site copy below matters.

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

- **Off-site copy (top priority).** The plan is to turn the retired storage node into a Proxmox Backup Server at a family member's house, seeded on the LAN and then synced over Tailscale. It would be the first copy of the guests outside the house, and the first real protection for the media dataset.
