# Network

## Topology

Flat `/24` home network. All devices on `10.0.0.0/24`. AdGuard Home handles DNS for the whole network.

```mermaid
graph TD
    ISP["ISP"] --> Router["Router\n10.0.0.254"]
    Router --> PVE1["pve-guide\n10.0.0.1"]
    Router --> PVE2["pve2\n10.0.0.2"]
    Router --> Devices["Other Devices"]
    PVE1 --> AG["AdGuard Home\nDNS: port 53"]
    AG -->|upstream| DNS["Cloudflare 1.1.1.1\nGoogle 8.8.8.8"]
    PVE1 --> NPM["Nginx Proxy Manager\n10.0.0.11"]
    NPM -->|SSL proxy| Services["Internal Services"]
    CF["Cloudflare\ntaylorsfunlab.com"] -->|DDNS| Router
    NPM -->|Let's Encrypt\nDNS challenge| CF
```

## DNS — AdGuard Home

- **Container:** CT 100 on pve-guide
- **DNS port:** 53
- **Web UI:** port 6060
- **Upstream DNS:** Cloudflare (1.1.1.1) + Google (8.8.8.8)
- **Function:** Network-wide ad and tracker blocking

All devices on the network use AdGuard as their DNS server (set via router DHCP).

## Reverse Proxy — Nginx Proxy Manager

- **Container:** CT 121 (portainer LXC) on pve-guide
- **IP:** 10.0.0.11

Handles all external access to services via `taylorsfunlab.com` subdomains. Wildcard SSL certificate (`*.taylorsfunlab.com`) issued via Let's Encrypt with Cloudflare DNS challenge — no ports exposed to the internet except 80/443.

### Proxy Hosts

| Subdomain | Backend | Port |
|---|---|---|
| `home.taylorsfunlab.com` | 10.0.0.11 | 7575 (Homarr) |
| `crafty.taylorsfunlab.com` | 10.0.0.11 | 8123 (Crafty) |
| `proxy.taylorsfunlab.com` | 10.0.0.11 | NPM itself |
| `proxmox.taylorsfunlab.com` | 10.0.0.1 | 8006 |
| `nc.taylorsfunlab.com` | 10.0.0.12 | 11000 (Nextcloud) |
| `radarr.taylorsfunlab.com` | 10.0.0.10 | 7878 |
| `sonarr.taylorsfunlab.com` | 10.0.0.10 | 8989 |

## Dynamic DNS — Cloudflare

Cloudflare DDNS container runs in the portainer LXC and keeps the `taylorsfunlab.com` A record pointed at the home IP. Updates automatically when the IP changes.

## VPN — Gluetun

All download traffic on the mediaServer VM is routed through a VPN via Gluetun. qBittorrent, NZBGet, and Prowlarr run with `network_mode: service:gluetun` — they have no direct internet access and cannot leak traffic if the VPN drops.

## Static IPs

Most containers use DHCP but the following have consistent IPs (either static or DHCP reservation):

| Host | IP | Notes |
|---|---|---|
| pve-guide | 10.0.0.1 | Static |
| pve2 | 10.0.0.2 | Static |
| portainer CT | 10.0.0.11 | DHCP reservation |
| media CT | 10.0.0.20 | Static |
| nextcloud CT | 10.0.0.12 | DHCP reservation |
| mediaServer VM | 10.0.0.10 | DHCP reservation |
