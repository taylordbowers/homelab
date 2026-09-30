# Portainer

Docker container management UI. Its LXC is also the lab's main Docker host for infrastructure services.

## Setup

- **Host:** portainer LXC (CT 121) on pve-guide, `10.0.0.11`
- **Web UI:** port 9443 (HTTPS)
- **Image:** `portainer/portainer-ce:lts`
- **LXC:** unprivileged Ubuntu 22.04, 3 cores, 24GB RAM, 24GB rootfs
- **Mounts:** the 22TB `tank_new` media dataset at `/data` (for Crafty's game data), plus `/dev/net/tun` for tunnel-capable containers

## Co-located Services

The portainer LXC runs these alongside Portainer:

| Service | Port | Page |
|---|---|---|
| Nginx Proxy Manager | 80 / 443 / 81 | [NPM](../network/nginx-proxy-manager.md) |
| Uptime Kuma | 3001 | [Uptime Kuma](uptime-kuma.md) |
| Homarr | 7575 | [Homarr](homarr.md) |
| Crafty Controller | 8443 / 8123 | [Crafty](../gaming/crafty.md) |
| Restreamer | 1935 / 8080 | [Restreamer](../network/restreamer.md) |
| Cloudflare DDNS | — | [Cloudflare DDNS](../network/cloudflare-ddns.md) |
| A few small private web apps | — | not documented publicly |

!!! note "Why this CT is still on pve-guide"
    CT 121 mounts `tank_new` directly, so it can't move to `pve` until the array does. That makes it the lab's main single point of failure for now, because monitoring and the reverse proxy both live here.

!!! warning "Backup gotcha"
    Docker's overlay2 storage means millions of small files, so this CT's nightly backup takes about 20 minutes. It once failed completely because host-owned files had been staged into the unprivileged CT. See [Backups → Lessons learned](../../infrastructure/backups.md#lessons-learned).

## Docker Compose

```yaml
portainer:
  image: portainer/portainer-ce:lts
  container_name: portainer
  restart: always
  ports:
    - 8000:8000
    - 9443:9443
  volumes:
    - /var/run/docker.sock:/var/run/docker.sock
    - portainer_data:/data
```
