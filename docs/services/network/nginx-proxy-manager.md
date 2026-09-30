# Nginx Proxy Manager

Reverse proxy with a web UI for managing proxy hosts and SSL certificates.

## Setup

- **Host:** portainer LXC (CT 121) on pve-guide, `10.0.0.11`
- **Admin UI:** port 81
- **HTTP:** port 80
- **HTTPS:** port 443

## SSL Certificates

Wildcard certificate for `*.taylorsfunlab.com` issued via Let's Encrypt using Cloudflare DNS challenge. This means no ports need to be open beyond 80/443 — DNS validation happens via the Cloudflare API.

The certificate renews automatically before it expires (Let's Encrypt certs last 90 days).

!!! tip "WebSockets through NPM"
    For apps that use WebSockets (like the Jarvis HUD voice stream), turn on *Websockets Support* on the proxy host and raise the timeouts. When testing with `curl`, force `--http1.1`: over HTTP/2 the upgrade headers are dropped and you get a misleading 404.

## Proxy Hosts

NPM fronts the internal services that need TLS: dashboards, the cloud suite, the *arr admin UIs, the hypervisor UI, and the Jarvis HUD. Backends on both nodes are reached by LAN IP. The specific hostnames and backends are deliberately not published here.

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
