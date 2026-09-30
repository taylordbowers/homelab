# Nextcloud AIO

Full self-hosted cloud suite — file storage, calendar, contacts, document editing, video calls, and more.

## Setup

- **Host:** nextcloud LXC (CT 101) on `pve`
- **IP:** 10.0.0.12
- **Container:** unprivileged LXC, Ubuntu 22.04, 3 cores / 4GB RAM, 40GB rootfs on `flash` (NVMe)
- **GPU:** RTX A3000 device nodes passed through (not actively used yet)
- **External access:** behind Nginx Proxy Manager with TLS
- **Install type:** Nextcloud All-in-One (AIO) via Docker

## Architecture

Nextcloud AIO manages its own container stack internally. All containers are deployed and updated through the AIO master container.

| Container | Role |
|---|---|
| `nextcloud-aio-mastercontainer` | AIO orchestrator (admin UI) |
| `nextcloud-aio-nextcloud` | Nextcloud PHP app |
| `nextcloud-aio-apache` | Web server (port 11000) |
| `nextcloud-aio-database` | PostgreSQL |
| `nextcloud-aio-redis` | Cache |
| `nextcloud-aio-imaginary` | Image processing |
| `nextcloud-aio-fulltextsearch` | Full-text search (Elasticsearch) |
| `nextcloud-aio-collabora` | Collabora Online — document editing |
| `nextcloud-aio-talk` | Nextcloud Talk — video calls |
| `nextcloud-aio-whiteboard` | Whiteboard app |
| `nextcloud-aio-notify-push` | Push notifications |

## Data Storage

User data lives on pve-guide's `tank_new` RAIDZ1 array. The `pve` host mounts the dataset over NFS at `/mnt/data`, and it's bind-mounted into the LXC as `/data`. The LXC rootfs and the AIO Docker volumes (database, config) are on `pve`'s `flash` pool.

!!! note
    Because data comes over NFS from the other node, pve-guide must be up for Nextcloud to serve files. The rootfs (including the database) is covered by the nightly vzdump backup. The user data on the array is not (see [Backups](../../infrastructure/backups.md#whats-excluded)).

## Deployment

The AIO master container is all you need to get started:

```bash
docker run -d \
  --name nextcloud-aio-mastercontainer \
  --restart always \
  -p 8080:8080 \
  -e APACHE_PORT=11000 \
  -e APACHE_IP_BINDING=0.0.0.0 \
  -v nextcloud_aio_mastercontainer:/mnt/docker-aio-config \
  -v /var/run/docker.sock:/var/run/docker.sock:ro \
  ghcr.io/nextcloud-releases/all-in-one:latest
```

Then access the AIO admin UI at `http://[server-ip]:8080` to configure and start all other containers.

## Nginx Proxy Manager Config

Proxied via NPM to `10.0.0.12:11000` with SSL. Required headers for Nextcloud:

```nginx
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
proxy_set_header X-Forwarded-Proto $scheme;
```
