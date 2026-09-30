# Filebrowser

A host-level web file manager that exposes each Proxmox node's full filesystem. It runs as a systemd service on each node, *not* in a container, so it can see every LXC subvolume on the ZFS pools.

## Setup

- **pve:** systemd service on the host, port 8085
- **pve-guide:** systemd service on the host, port 8085
- **Root path:** `/` (full filesystem)

Recent Filebrowser releases require an admin password of at least 12 characters, which is worth knowing when setting it up on a new node.

## Why host-level instead of an LXC

Inside an LXC, Filebrowser would only see that LXC's filesystem. On the host it can browse:

- Every container's rootfs (under `/flash/subvol-NNN-disk-0/...`)
- The bulk ZFS array on pve-guide (`tank_new`)
- Proxmox config files (`/etc/pve/...`)

That makes it a single file UI for everything, without SSHing into each node.

## Security

Filebrowser has *full* host access, so treat it that way. It should:

- Be reachable only from the LAN (no external proxy)
- Use a strong admin password
- Not be linked from public docs or the dashboard

!!! warning "Unprivileged containers and file ownership"
    Files you create inside an **unprivileged** CT's subvolume from the host (through Filebrowser or a shell) end up owned by host root (uid 0). That shows up as `nobody` inside the CT, and it can break that CT's backups. Fix ownership to the CT's mapped uid (usually `100000`) afterwards. See [Backups → Lessons learned](../../infrastructure/backups.md#lessons-learned).

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
