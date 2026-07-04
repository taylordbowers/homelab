# Filebrowser

Host-level web file manager exposing the full Proxmox node filesystem. Runs as a systemd service on each node — *not* containerized — so it has visibility into every LXC subvolume on the underlying ZFS pool.

## Setup

- **Pve-guide:** systemd service on the host, port 8085
- **Pve2:** systemd service on the host, port 8085
- **Root path:** `/` (full filesystem)

## Why host-level instead of an LXC

Running Filebrowser inside an LXC would give it visibility only into that LXC's view of the filesystem. Running it on the host means it can browse:

- Every container's rootfs (under `/flash/subvol-NNN-disk-0/...`)
- The bulk ZFS array on pve-guide (`tank_new`)
- Proxmox config files (`/etc/pve/...`)

This makes it a useful "everything available everywhere" file UI without needing to SSH onto each node.

## Security

Filebrowser has *full* host access — treat it accordingly. It should:

- Be reachable only from the LAN (no external proxy)
- Use a strong admin password
- Not be linked from public docs or the dashboard

## Systemd unit (sketch)

```ini
[Unit]
Description=Filebrowser
After=network.target

[Service]
ExecStart=/usr/local/bin/filebrowser -d /etc/filebrowser/db.db -r / -p 8085 -a 0.0.0.0
Restart=on-failure

[Install]
WantedBy=multi-user.target
```
