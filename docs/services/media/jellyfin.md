# Jellyfin

Personal media server. Streams movies, TV shows, and music to any device.

## Setup

- **Host:** CT 103 (jellyfin LXC) on pve2
- **IP:** 10.0.0.14
- **Port:** 8096
- **URL:** `stream.taylorsfunlab.com`
- **GPU:** NVIDIA GTX 980 Ti via LXC device passthrough (shared with CT 101 and CT 104)

## Container Config

- **Type:** Privileged LXC, Ubuntu 24.04
- **CPU:** 4 cores
- **RAM:** 8 GB
- **Disk:** 64 GB (flash ZFS)
- **Features:** `nesting=1` (Docker-in-LXC)
- **Autostart:** `systemd: jellyfin-stack.service`

## Services in This Stack

| Service | Port | Description |
|---------|------|-------------|
| Jellyfin | 8096 | Media server |
| Jellyseerr | 5055 | Movie/TV request management — connected to Radarr & Sonarr on VM 119 |
| Jellystat | 3000 | Usage statistics dashboard (requires initial setup via web UI) |
| Jellystat-DB | — | postgres 15.2 backend for Jellystat |

## Docker Compose (key sections)

```yaml
jellyfin:
  image: lscr.io/linuxserver/jellyfin:latest
  runtime: nvidia
  environment:
    - PUID=1000
    - PGID=993
    - TZ=America/Chicago
    - NVIDIA_VISIBLE_DEVICES=all
    - NVIDIA_DRIVER_CAPABILITIES=compute,video,utility
    - JELLYFIN_PublishedServerUrl=http://10.0.0.14
  volumes:
    - /docker/jellyfin/config:/config
    - /data:/data
  ports:
    - 8096:8096
  restart: unless-stopped

jellyseerr:
  image: fallenbagel/jellyseerr:latest
  environment:
    - TZ=America/Chicago
  volumes:
    - /docker/jellyfin/jellyseerr:/app/config
  ports:
    - 5055:5055
  restart: unless-stopped

jellystat:
  image: cyfershepard/jellystat:latest
  environment:
    - POSTGRES_USER=postgres
    - POSTGRES_PASSWORD=mypassword
    - POSTGRES_IP=jellystat-db
    - POSTGRES_PORT=5432
    - JWT_SECRET=my-secret-jwt-key
    - TZ=America/Chicago
  ports:
    - 3000:3000
  depends_on:
    - jellystat-db
  restart: unless-stopped
```

## Hardware Transcoding

The GTX 980 Ti is shared via LXC device bind-mounts. `nvidia-container-toolkit` is installed inside the container and Docker is configured with the `nvidia` runtime. NVENC confirmed working:

- `h264_nvenc` ✓
- `hevc_nvenc` ✓
- `av1_nvenc` ✓

See [GPU Passthrough](../../infrastructure/gpu-passthrough.md) for full LXC config details.

## Data Mounts

| Mount | Source | Destination |
|-------|--------|-------------|
| Media files | `//10.0.0.20/data` (CIFS from CT 200) | `/data` |
| pve-guide tank | `10.0.0.1:/tank_new/subvol-200-disk-0` (NFS) | `/mnt/guide` |

## Media Library

| Library | Path |
|---------|------|
| Movies | `/data/Movies` (150+ titles) |
| TV Shows | `/data/Shows` (20+ series) |
| Music | `/data/Music` |