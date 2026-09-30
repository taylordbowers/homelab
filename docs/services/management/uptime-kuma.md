# Uptime Kuma

Service uptime monitoring and alerting. It watches every running service in the lab, and its alerts drive the [self-healing](../automation/self-healing.md) pipeline.

## Setup

- **Host:** portainer LXC (CT 121) on pve-guide, `10.0.0.11`
- **Port:** 3001
- **Image:** `louislam/uptime-kuma:1`
- **Compose location:** `/opt/uptime-kuma/docker-compose.yml`
- **Monitors:** 43, all active

## Alerting

| Channel | Used for |
|---|---|
| **Discord** (`#homelab-alerts`) | Primary. Every UP/DOWN goes here, and it's the default for new monitors |
| **Email** | Only for game-server monitors |
| **Self-heal webhook** | Every monitor. Sends the alert to the [self-healing receiver](../automation/self-healing.md) on CT 102 |

## Push monitors (dead-man's switch)

Scheduled jobs that have no port to probe, such as the trading bot's sessions, **push** a heartbeat to Kuma when they finish. If a push doesn't arrive within the interval, Kuma alerts. That catches jobs that silently stop running, not just ones that crash.

## Coverage check

The weekly health audit compares every running guest's IP with the targets of all Kuma monitors and flags any guest with no monitor at all. Guests that are unmonitored by design (the Windows VM) are on an ignore list.

## Why this and not Grafana?

Grafana answers "how is my stuff *performing* over time", but it needs a metrics pipeline (Prometheus + exporters on each node). Uptime Kuma answers "is my stuff *up*". It checks services, alerts when they go down, and tracks response time and SSL expiry. It set up in minutes, and Homarr already covers at-a-glance status.

## Docker Compose

```yaml
services:
  uptime-kuma:
    image: louislam/uptime-kuma:1
    container_name: uptime-kuma
    restart: unless-stopped
    dns:
      - 10.0.0.5   # internal DNS (AdGuard), needed to resolve internal hostnames
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
    The custom `dns:` block is required. By default Docker uses public resolvers, which can't see internal hostnames (such as anything AdGuard rewrites). Without it, checks on LAN-only names fail with `ENOTFOUND`.

## Monitor types in use

- **HTTP(s)**: most web services. `accepted_statuscodes: ["200-299","301","302"]` so login redirects don't trip alerts.
- **Ping**: Proxmox hosts and guests that allow ICMP.
- **TCP port**: services without an HTTP endpoint.
- **Push**: scheduled jobs (see above).

## Tips learned the hard way

- **Self-signed TLS endpoints** (Proxmox UI, Portainer, Crafty) need `ignoreTls: true`, or every check fails on cert validation.
- **NPM-fronted services** that redirect need `301`/`302` in `accepted_statuscodes`, or `maxredirects: 0` so the redirect itself counts as up.
- **Apps mounted at a path** (Guacamole at `/guacamole/`) need the full URL including the path, because `/` returns 404.
- **Kuma lives on pve-guide.** When that node was offline during the house move, monitoring and push heartbeats were down with it. Moving CT 121 to `pve` is on the roadmap.
