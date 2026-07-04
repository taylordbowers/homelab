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

NPM fronts the internal services that need TLS — dashboards, the cloud suite, the *arr stack admin UIs, and the hypervisor UI. Specific hostnames and backends are intentionally not published here.

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
