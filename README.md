# Taylor's Homelab

A two-node Proxmox VE cluster running a full self-hosted stack — media, photos, cloud storage, game servers, and more.

**[View the full docs →](https://taylordbowers.github.io/homelab)**

## What's Running

| Service | Role |
|---|---|
| Jellyfin | Media server with GPU transcoding |
| Immich | Photo library with AI facial recognition (GPU) |
| Nextcloud AIO | Cloud storage, docs, and video calls |
| Sonarr / Radarr / Lidarr | Automated media management |
| Prowlarr / Bazarr / Profilarr | Indexer and subtitle management |
| qBittorrent + NZBGet | Download clients (VPN-routed via Gluetun) |
| AdGuard Home | Network-wide DNS ad blocking |
| Nginx Proxy Manager | Reverse proxy + SSL for taylorsfunlab.com |
| Crafty Controller | Minecraft server management |
| Portainer + Homarr | Container management + dashboard |

## Hardware

| Node | CPU | RAM | Storage | GPU |
|---|---|---|---|---|
| pve-guide | Xeon E3-1245 v3 | 32GB | 932GB SSD + 4x 7.3TB RAIDZ1 | Intel iGPU |
| pve2 | i7-4790 | 32GB | 932GB SSD | GTX 980 Ti |

## Docs

Documentation is built with [MkDocs Material](https://squidfunk.github.io/mkdocs-material/).

```bash
pip install mkdocs-material
mkdocs serve        # local preview
mkdocs gh-deploy    # deploy to GitHub Pages
```
