# Nginx Proxy Manager

Reverse proxy with a web UI for managing proxy hosts and SSL certificates.

## Setup

- **Host:** portainer LXC (CT 121) on pve-guide, `10.0.0.11`
- **Admin UI:** port 81
- **HTTP:** port 80
- **HTTPS:** port 443

## SSL Certificates

Wildcard certificate for `*.taylorsfunlab.com` issued via Let's Encrypt using Cloudflare DNS challenge. This means no ports need to be open beyond 80/443 — DNS validation happens via the Cloudflare API.

Certificate renews automatically every 90 days.

## Proxy Hosts

| Domain | Backend | Notes |
|---|---|---|
| `home.taylorsfunlab.com` | 10.0.0.11:7575 | Homarr dashboard |
| `crafty.taylorsfunlab.com` | 10.0.0.11:8123 | Crafty Controller |
| `nc.taylorsfunlab.com` | 10.0.0.12:11000 | Nextcloud AIO |
| `proxmox.taylorsfunlab.com` | 10.0.0.1:8006 | Proxmox UI (HTTPS backend) |
| `radarr.taylorsfunlab.com` | 10.0.0.10:7878 | Radarr |
| `sonarr.taylorsfunlab.com` | 10.0.0.10:8989 | Sonarr |

## Docker Compose

```yaml
nginx-proxy-manager:
  image: jc21/nginx-proxy-manager:latest
  container_name: nginx-proxy-manager
  restart: unless-stopped
  ports:
    - 80:80
    - 81:81
    - 443:443
  volumes:
    - ./data:/data
    - ./letsencrypt:/etc/letsencrypt
```
