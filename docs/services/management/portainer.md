# Portainer

Docker container management UI. Also hosts several other services in the same LXC.

## Setup

- **Host:** portainer LXC (CT 121) on pve-guide, `10.0.0.11`
- **Web UI:** port 9443 (HTTPS)
- **Image:** `portainer/portainer-ce:lts`
- **Resources:** 3 cores, 24GB RAM

## Co-located Services

The portainer LXC runs several other services alongside Portainer:

- Nginx Proxy Manager
- Homarr
- Crafty Controller
- Restreamer
- Cloudflare DDNS

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
