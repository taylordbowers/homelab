# Cloudflare DDNS

Keeps the `taylorsfunlab.com` DNS A record updated when the home IP changes.

## Setup

- **Host:** portainer LXC (CT 121) on pve-guide
- **Image:** `favonia/cloudflare-ddns:latest`

## Docker Compose

```yaml
cloudflare-ddns:
  image: favonia/cloudflare-ddns:latest
  container_name: cloudflare-ddns
  restart: always
  environment:
    - CLOUDFLARE_API_TOKEN=your_token_here
    - DOMAINS=taylorsfunlab.com
    - PROXIED=true
```

Uses a scoped Cloudflare API token with `Zone:DNS:Edit` permissions only. Set `PROXIED=true` to route traffic through Cloudflare's proxy (hides home IP).
