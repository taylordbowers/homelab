# Taylor's Homelab

A two-node Proxmox VE cluster running a full self-hosted stack: media, photos, cloud storage, and game servers. An in-cluster Claude Code agent monitors it, fixes what it's allowed to fix, and documents everything.

**[View the full docs →](https://taylordbowers.github.io/homelab)**

## What's Running

| Service | Role |
|---|---|
| Jellyfin + Jellyseerr + Jellystat | Media server with NVENC transcoding (RTX A3000) |
| Immich | Photo library with facial recognition + smart search |
| Nextcloud AIO | Cloud storage, docs, and video calls |
| Sonarr / Radarr / Lidarr | Automated media management |
| Prowlarr / Bazarr / Profilarr / Sportarr | Indexers, subtitles, quality profiles, sports |
| qBittorrent + NZBGet | Download clients (VPN-routed via Gluetun) |
| AdGuard Home | DNS ad blocking + internal DNS rewrites |
| Nginx Proxy Manager | Reverse proxy + wildcard SSL |
| Tailscale | Remote access (subnet router + exit node) |
| Uptime Kuma | 42 monitors, alerts to Discord + email |
| Apache Guacamole | Browser-based RDP/VNC gateway |
| Crafty Controller + AMP | Minecraft + multi-game servers |
| Portainer + Homarr | Container management + dashboard |
| Claude Code agent | AI cluster operator via Proxmox MCP, with self-healing, a Discord agent, and a voice HUD |
| Trading bots | Two paper-trading research bots |

## Hardware

| Node | Role | CPU | RAM | Storage | GPU |
|---|---|---|---|---|---|
| pve | Primary (laptop) | i9-11950H | 64GB | 1TB NVMe (`flash` ZFS) | RTX A3000 Laptop 6GB |
| pve-guide | Storage | Xeon E3-1245 v3 | 32GB | 4x 8TB RAIDZ1 (`tank_new`) + 1TB `flash` | — |

Every guest is backed up nightly to the RAIDZ1 array (7 daily / 4 weekly / 3 monthly).

## Docs

Documentation is built with [MkDocs Material](https://squidfunk.github.io/mkdocs-material/) and deployed to GitHub Pages by GitHub Actions on every push to `main`.

```bash
pip install mkdocs-material
mkdocs serve        # local preview
```
