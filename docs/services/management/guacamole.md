# Apache Guacamole

Clientless remote desktop gateway — RDP, VNC, and SSH sessions in a browser tab. Backed by a small Docker stack.

## Setup

- **Host:** guacamole LXC (CT 105) on pve2, `10.0.0.x`
- **Port:** 8080 (path: `/guacamole/`)
- **Components:**
    - `guacamole/guacamole` — the web app (Tomcat servlet)
    - `guacamole/guacd` — the protocol daemon (RDP/VNC/SSH backends)
    - `mariadb:11` — auth + connections database

## Container Config

- **Type:** Privileged LXC, Debian
- **CPU:** 2 cores
- **RAM:** 2 GB

## Primary use case

Used to reach the [Windows 11 VM](../../hardware/pve2.md) (VM 301) over RDP from any browser without installing an RDP client. Sessions are recorded by Guacamole if the recording feature is enabled.

## Notes

- The Guacamole web app is mounted at `/guacamole/` not `/` — hitting `/` returns 404.
- The MariaDB schema gets seeded on first run via the official init scripts; do not destroy the DB volume without exporting connection definitions first.
