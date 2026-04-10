# Taylor's Homelab

A two-node Proxmox VE cluster running a full self-hosted stack — media, photos, cloud storage, game servers, and more. Built around the philosophy of owning your own data and services.

## Cluster Overview

```mermaid
graph TD
    Internet["🌐 Internet"] -->|Cloudflare DDNS| Router["🔀 Router\n10.0.0.254"]
    Router --> PVE1["🖥️ pve-guide\n10.0.0.1\nXeon E3-1245 v3 / 32GB"]
    Router --> PVE2["🖥️ pve2\n10.0.0.2\ni7-4790 / 32GB / GTX 980 Ti"]

    PVE1 --> AG["AdGuard Home\nDNS + Ad Blocking"]
    PVE1 --> NPM["Nginx Proxy Manager\n+ Homarr + Crafty + Restreamer"]
    PVE1 --> MEDIA["Media LXC\n22TB RAIDZ1 Data Store"]
    PVE1 --> ARR["mediaServer VM\nSonarr · Radarr · Lidarr\nBazarr · Prowlarr + VPN"]

    PVE2 --> NC["Nextcloud AIO\n10.0.0.12"]
    PVE2 --> JF["Jellyfin VM\nJellyfin · Jellyseerr\nImmich + GPU ML"]
    PVE2 --> CLAUDE["Claude Code MCP\n10.0.0.13"]
```

## Quick Reference

| Service | URL | Host |
|---|---|---|
| Proxmox UI | `proxmox.taylorsfunlab.com` | pve-guide |
| Dashboard | `home.taylorsfunlab.com` | portainer CT |
| Jellyfin | `stream.taylorsfunlab.com` | jellyfin VM |
| Jellyseerr | port 5055 | jellyfin VM |
| Immich | port 2283 | jellyfin VM |
| Nextcloud | `nc.taylorsfunlab.com` | nextcloud CT |
| Sonarr | `sonarr.taylorsfunlab.com` | mediaServer VM |
| Radarr | `radarr.taylorsfunlab.com` | mediaServer VM |
| AdGuard | port 6060 | adguard CT |
| Portainer | `proxy.taylorsfunlab.com` | portainer CT |
| Crafty | `crafty.taylorsfunlab.com` | portainer CT |

## IP Scheme

| Host | IP |
|---|---|
| Router | 10.0.0.254 |
| pve-guide | 10.0.0.1 |
| pve2 | 10.0.0.2 |
| adguard CT | DHCP |
| portainer CT | 10.0.0.11 |
| media CT | 10.0.0.20 |
| mediaServer VM | 10.0.0.10 |
| nextcloud CT | 10.0.0.12 |
| claude CT | 10.0.0.13 |
| jellyfin VM | DHCP |

## What's Running

### Media
- **Jellyfin** — media server with hardware transcoding
- **Immich** — Google Photos replacement with GPU-accelerated facial recognition (GTX 980 Ti)
- **Jellyseerr** — media request management
- **Jellystat** — Jellyfin analytics
- **Sonarr / Radarr / Lidarr** — automated media management
- **Bazarr** — subtitle management
- **Prowlarr** — indexer aggregation
- **Profilarr** — quality profile management
- **Sportarr** — sports content management
- **qBittorrent + NZBGet** — download clients (VPN-routed via Gluetun)

### Cloud & Productivity
- **Nextcloud AIO** — full cloud suite with Collabora, Talk, Fulltextsearch, Whiteboard

### Network
- **AdGuard Home** — network-wide DNS ad blocking
- **Nginx Proxy Manager** — reverse proxy with Let's Encrypt SSL
- **Cloudflare DDNS** — dynamic DNS for `taylorsfunlab.com`

### Management
- **Proxmox VE** — hypervisor cluster management
- **Portainer** — Docker container management
- **Homarr** — homelab dashboard
- **Claude Code MCP** — AI-assisted cluster management

### Gaming
- **Crafty Controller** — Minecraft server management (Java + Bedrock)
- **AMP** — multi-game server manager (stopped)
