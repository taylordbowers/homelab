# Homarr

Homelab dashboard — shows all services, system stats, and quick links in one place.

## Setup

- **Host:** portainer LXC (CT 121) on pve-guide, `10.0.0.11`
- **Port:** 7575
- **URL:** `home.taylorsfunlab.com`
- **Image:** `ghcr.io/homarr-labs/homarr:latest`

## Docker Compose

```yaml
homarr:
  image: ghcr.io/homarr-labs/homarr:latest
  container_name: homarr
  restart: unless-stopped
  ports:
    - 7575:7575
  volumes:
    - /var/run/docker.sock:/var/run/docker.sock
    - /home/taylor/homarr/appdata:/appdata
```
