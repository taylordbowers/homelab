# Cloudflare DDNS

Keeps the `taylorsfunlab.com` DNS A record updated when the home IP changes.

## Setup

- **Host:** portainer LXC (CT 121) on pve-guide
- **Records managed:** the apex domain only. Internal-only services have **no** public records; they resolve through AdGuard rewrites on the LAN
- **Image:** `favonia/cloudflare-ddns:latest`

## Docker Compose

```yaml
cloudflare-ddns:
  image: favonia/cloudflare-ddns:latest
  container_name: cloudflare-ddns
  restart: always
  environment:
    - CLOUDFLARE_API_TOKEN=${CLOUDFLARE_API_TOKEN}
    - DOMAINS=taylorsfunlab.com
    - PROXIED=true
```

Uses a scoped Cloudflare API token with `Zone:DNS:Edit` permissions only. Set `PROXIED=true` to route traffic through Cloudflare's proxy (hides home IP).
