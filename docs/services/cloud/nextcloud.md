# Nextcloud AIO

Full self-hosted cloud suite — file storage, calendar, contacts, document editing, video calls, and more.

## Setup

- **Host:** nextcloud LXC (CT 101) on pve2
- **IP:** 192.168.1.141
- **URL:** `nc.taylorsfunlab.com`
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

Nextcloud data is stored on the `tank_new` RAIDZ1 array via a bind mount from the host (`/mnt/data → /data` inside the LXC). The LXC rootfs is on `flash` (40GB).

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

NPM proxies `nc.taylorsfunlab.com` → `192.168.1.141:11000` with SSL. Required headers for Nextcloud:

```nginx
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
proxy_set_header X-Forwarded-Proto $scheme;
```
