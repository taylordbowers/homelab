# Apache Guacamole

A clientless remote-desktop gateway: RDP, VNC, and SSH sessions in a browser tab, backed by a small Docker stack.

## Setup

- **Host:** guacamole LXC (CT 105) on `pve`, `10.0.0.16`
- **Port:** 8080 (path: `/guacamole/`)
- **Compose:** `/opt/guacamole/`, started by a `guacamole.service` systemd unit
- **Components:**
    - `guacamole/guacamole`: the web app (Tomcat servlet)
    - `guacamole/guacd`: the protocol daemon (RDP/VNC/SSH backends)
    - `mariadb:11`: auth and connections database

## Container Config

- **Type:** Privileged LXC, Ubuntu 24.04
- **CPU:** 2 cores
- **RAM:** 2 GB
- **Disk:** 10 GB (`flash` ZFS)

## Primary use case

Reaching the [Windows 11 VM](../../hardware/win11-vm.md) (VM 301) over RDP from any browser, with no RDP client to install.

## Notes

- The web app is mounted at `/guacamole/`, not `/`. Hitting `/` returns 404, which matters for uptime checks.
- The MariaDB schema is seeded on first run by the official init scripts. Don't destroy the DB volume without exporting the connection definitions first.
