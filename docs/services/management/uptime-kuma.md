# Uptime Kuma

Service uptime monitoring + alerting. Tracks every running service in the homelab and surfaces problems before they're noticed elsewhere.

## Setup

- **Host:** portainer LXC (CT 121) on pve-guide, `10.0.0.11`
- **Port:** 3001
- **Image:** `louislam/uptime-kuma:1`
- **Compose location:** `/opt/uptime-kuma/docker-compose.yml`

## Why this and not Grafana?

Grafana is a metrics + visualization platform — it answers "how is my stuff *performing* over time" but needs a data source (Prometheus + node_exporter on each Proxmox node, set up per-service). Uptime Kuma is the lighter complement that answers "is my stuff *up*" — pings services, alerts when they go down, tracks response time and SSL expiry. For a homelab where Homarr already covers at-a-glance status, Uptime Kuma fills the gap Homarr doesn't (alerting + historical uptime), and it deploys in minutes vs hours.

## Docker Compose

```yaml
services:
  uptime-kuma:
    image: louislam/uptime-kuma:1
    container_name: uptime-kuma
    restart: unless-stopped
    dns:
      - 10.0.0.x   # internal DNS (AdGuard) — needed to resolve internal hostnames
      - 1.1.1.1    # public fallback
    ports:
      - "3001:3001"
    volumes:
      - uptime-kuma-data:/app/data
    healthcheck:
      test: ["CMD", "extra/healthcheck"]
      interval: 60s
      timeout: 30s
      retries: 5

volumes:
  uptime-kuma-data:
```

!!! note
    The custom `dns:` block is required. By default Docker uses public resolvers, which can't see internal hostnames (e.g. anything that AdGuard rewrites). Without this, Kuma's checks fail with `ENOTFOUND` on services that resolve only via the LAN's DNS.

## Monitor types in use

- **HTTP(s)** — most web services. `accepted_statuscodes: ["200-299","301","302"]` so login redirects don't trip alerts.
- **Ping** — Proxmox hosts and a few VMs (where ICMP is allowed).
- **TCP port** — Windows 11 VM (3389/RDP), since Windows Firewall blocks ICMP by default.

## Tips learned the hard way

- **Self-signed TLS endpoints** (Proxmox UI, Portainer, Crafty) need `ignoreTls: true` or every check fails on cert validation.
- **NPM-fronted services** that issue redirects need either `accepted_statuscodes` to include `301`/`302`, or `maxredirects: 0` so the redirect itself counts as up.
- **Apps mounted at a path** (e.g. Guacamole at `/guacamole/`) need the full URL with the path — `/` returns 404.
